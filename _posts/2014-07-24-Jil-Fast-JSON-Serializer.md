# 分享一个快速的Json（反）序列化开源项目 Jil

> 原文：https://www.cnblogs.com/tansm/p/3865975.html

我们不缺少JSON的序列化库，但我们缺少一个性能非常好的库，这对于网站来说非常重要。今天我发现了Jil。

他是开源的代码： [https://github.com/kevin-montrose/Jil](https://github.com/kevin-montrose/Jil)

在他主页上有详细的介绍其性能的表现，我这里就不转述了，他最重要的特点就是性能，Emit那当然不会少了，当想让他超越其他的库光一个Emit肯定不行，他还有很多其他的优化。

- 公共缓冲区

为介绍GC的压力，他使用了诸如builder.CommonCharBuffer这样的功能缓冲，builder.CommonStringBuffer也是这样的应用。

- 内联

很多的方法，都标记了[MethodImpl(MethodImplOptions.AggressiveInlining)]以便编译器尽可能的内联。

- 减少计算

比如将int转换为string，原先的代码是这样写的。

```text
1         [MethodImpl(MethodImplOptions.AggressiveInlining)]
 2         static void _CustomWriteInt(TextWriter writer, int number, char[] buffer)
 3         {
 4             // Gotta special case this, we can't negate it
 5             if (number == int.MinValue)
 6             {
 7                 writer.Write("-2147483648");
 8                 return;
 9             }
10
11             var ptr = InlineSerializer<object>.CharBufferSize - 1;
12
13             var copy = number;
14             if (copy < 0)
15             {
16                 copy = -copy;
17             }
18
19             do
20             {
21                 var ix = copy % 10;
22                 copy /= 10;
23
24                 buffer[ptr] = (char)('0' + ix);
25                 ptr--;
26             } while (copy != 0);
27
28             if (number < 0)
29             {
30                 buffer[ptr] = '-';
31                 ptr--;
32             }
33
34             writer.Write(buffer, ptr + 1, InlineSerializer<object>.CharBufferSize - 1 - ptr);
35         }
```

是不是觉得已经很好了，但是他们还不断进取，改成这样：

```text
1         [MethodImpl(MethodImplOptions.AggressiveInlining)]
 2         static void _CustomWriteInt(TextWriter writer, int number, char[] buffer)
 3         {
 4             var ptr = InlineSerializer<object>.CharBufferSize - 1;
 5
 6             uint copy;
 7             if (number >= 0)
 8                 copy = (uint)number;
 9             else
10             {
11                 writer.Write('-');
12                 copy = 1 + (uint)~number;
13             }
14
15             do
16             {
17                 var ix = copy % 100;
18                 copy /= 100;
19
20                 var chars = DigitPairs[ix];
21                 buffer[ptr--] = chars.Second;
22                 buffer[ptr--] = chars.First;
23             } while (copy != 0);
24
25             if (buffer[ptr + 1] == '0')
26                 ++ptr;
27
28             writer.Write(buffer, ptr + 1, InlineSerializer<object>.CharBufferSize - 1 - ptr);
29         }
```

其中，那个DigitPairs是什么呢？

```text
1 struct TwoDigits
 2         {
 3             public readonly char First;
 4             public readonly char Second;
 5
 6             public TwoDigits(char first, char second)
 7             {
 8                 First = first;
 9                 Second = second;
10             }
11         }
12
13         private static readonly TwoDigits[] DigitPairs;
14
15         static Methods()
16         {
17             DigitPairs = new TwoDigits[100];
18             for (var i=0; i < 100; ++i)
19                 DigitPairs[i] = new TwoDigits((char)('0' + (i / 10)), (char)+('0' + (i % 10)));
20         }
```

 

是不是想法很高呢？

 

- 减少判断

原先的方法是这样的

```text
1 static bool IsWhiteSpace(int c)
 2         {
 3             // per http://www.ietf.org/rfc/rfc4627.txt
 4             // insignificant whitespace in JSON is defined as
 5             //  \u0020  - space
 6             //  \u0009  - tab
 7             //  \u000A  - new line
 8             //  \u000D  - carriage return
 9
10             return
11                 c == 0x20 ||
12                 c == 0x09 ||
13                 c == 0x0A ||
14                 c == 0x0D;
15         }
```

可以这样改，哈哈，其实我想这么改，不知道对不对？因为我觉得大多数情况下不是空白，所以一直要判断4次才能返回，而我改成这样：

```text
1 static bool IsWhiteSpace(int c)
 2         {
 3             // per http://www.ietf.org/rfc/rfc4627.txt
 4             // insignificant whitespace in JSON is defined as
 5             //  \u0020  - space
 6             //  \u0009  - tab
 7             //  \u000A  - new line
 8             //  \u000D  - carriage return
 9
10             return
11                 c < 0x21 && (
12                 c == 0x20 ||
13                 c == 0x09 ||
14                 c == 0x0A ||
15                 c == 0x0D);
16         }
```
