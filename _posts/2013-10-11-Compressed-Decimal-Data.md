# 压缩十进制数据的一次实践

> 原文：https://www.cnblogs.com/tansm/p/WriteReadDecimalDemo.html

# 简介

在ERP应用中，经常要将大批量的数据下载到客户端本地，例如查询、报表或导出等功能，这将消耗大量的网络资源来传输数据，特别是在移动办公时，较大的数据量将造成等待时间太长。本文以压缩十进制数据为例描述了压缩这些数据的实践。

# 分析案例

在传输的数据中，有各种类型的数据，有字符串、时间、十进制或bool等，本文将分析十进制这种类型。

在.net中，decimal是一个可以描述较大数据和小数，且保证计算准确的数据类型，比较适合ERP应用，但是他占用的空间比较大，反编译其申明可以看到其占用4个int32，即高达16个字节。

```text
1     private int flags;
2     private int hi;
3     private int lo;
4     private int mid;
```

直接调用[GetBits(Decimal) : Int32[]](http://msdn.microsoft.com/zh-cn/library/vstudio/system.decimal.getbits.aspx)存储显然不合算。

# 参考实现

微软内部是如何存储的呢？[System.IO.BinaryWriter](http://msdn.microsoft.com/zh-cn/library/vstudio/system.io.binarywriter.aspx)的默认实现还真的是这么干的，请看：

```text
1 public virtual void Write(decimal value)
2 {
3     decimal.GetBytes(value, this._buffer);
4     this.OutStream.Write(this._buffer, 0, 0x10);
5 }
```

注:不过他有点耍赖,调用了internal的GetBytes减少了byte[]数组的不断创建。

在二进制序列化的实现System.Runtime.Serialization.Formatters.Binary.__BinaryWriter中,使用了较为巧妙的方法,他将其转换为字符串存储:

```text
1 internal void WriteDecimal(decimal value)
2 {
3     this.WriteString(value.ToString(CultureInfo.InvariantCulture));
4 }
```

通常情况下,一列的数据都是很小的数字，例如数量或者单价,可能0,也可能正整数,也可能是0.17这样的小数.根据[WriteString](http://msdn.microsoft.com/zh-cn/library/vstudio/yzxa6408.aspx)的实现可知，需要至少一个字节描述字符串的长度，然后一个字符占用一个字节（默认utf8），也就是说，数字0占用2个字节，0.17占用5个字节。

还不错啊，至少比16个字节好多了。

# 有没有更好的压缩

## 观察

我对微软的实现还不够满意，仔细观察常见的数据，可以总结道：

1、  通常一个小数由整数部分、小数部分和小数位数组成，例如0.17整数是0，小数部分是17，小数位数是2，另外一个例子1.007，整数部分是1，小数部分是7，小数位数是3；

2、  如果小数位数是0，即整数，那么就不必描述小数部分了；

3、  如果数字就是0，这种情况非常多，是不是用1个字节而不是2个字节描述呢？

4、  像17这样的数字，用字符串描述就需要2个字节，而如果用二进制，1个字节就够了。

## 基本方法

所以我的方法是：第一个字节描述小数位数，后面的字节描述整数部分，如果小数位数大于0，则还包含小数部分的整数描述。

例如0.12，我会存储2，0，12三个正整数，十六进制描述就是

**02**

**00**

**0C**

** **

** **

** **

** **

** **

** **

** **

我在上面多处提到正整数，我们可以利用[**7BitEncodedInt**](http://msdn.microsoft.com/zh-cn/library/vstudio/system.io.binarywriter.write7bitencodedint.aspx)方式填写整数部分和小数部分，他是一种可变长的描述方法。

如果数字是整数，我们还可以省略小数部分，例如12

**00**

**0C**

** **

** **

** **

** **

** **

** **

** **

** **

到这里，我们发现一个问题，如果是数字0，第一个字节和上面是一样的（为0），这样即使0也还需要再存储一个字节0，用来确定整数部分是0，不然读取时，就无法区分0和12了，但这就达不到我之前提到的数字0使用一个字节来描述的要求。

而且，这里还有负数的问题，还有如果decimal的值大于int32的范围怎么办？所以我在第一个字节上做了手脚，把他再拆分成4个部分。

**A**

**A**

**A**

**A**

**A**

**B**

**C**

**D**

第一个字节的8个位中，前5位存储小数位数（A区），最大值是31，一般小数位数不会超过这个数字（事实上，下面的算法最多仅9位）。

B区一个位，如果是0，表示当前数字是0，其他任何位都无需考虑了，也没有整数部分和小数部分，如果是1，表示此值非0，通过这种方法，我们就实现了数字0仅用一个字节描述的目的。

C区一个位，如果是0表示正数，如果是1表示此值为负数；

D区一个位，如果是0表示此值使用压缩的存储方法，1表示此值太大或太小，使用了16个字节的原样输出。

现在还是0.12为例，其第一个字节其二进制实际上是：

**0**

**0**

**0**

**1**

**0**

**1**

**0**

**0**

第一个字节换算成十六进制为：0x14，所以0.12其存储的内容是：

**14**

**00**

**0C**

** **

** **

** **

** **

** **

** **

** **

## 参考实现

下面是写入时的代码，我使用了比较笨拙的办法获取小数部分的整数描述值。

```text
1 // Fast access for 10^n where n is 0-9
 2 private static UInt32[] Powers10 = new UInt32[] {
 3     10,
 4     100,
 5     1000,
 6     10000,
 7     100000,
 8     1000000,
 9     10000000,
10     100000000,
11     1000000000
12 };
13
14 private static void WriteToFullByte(SerializeContext context, decimal[] values) {
15     //整个逻辑放在一个方法中，目的是防止foreach循环中不断跳入其他方法，增加消耗，在这个性能要求较高的程序是允许的。
16     var stream = context.Stream;
17     int flag;
18     int a, b, c;
19     decimal d, e;
20
21     foreach (var value in values) {
22         if (value == 0m) {
23             //当数字是0时，第一个字节的所有位都是0，直接写0
24             stream.WriteByte(byte.MinValue);
25         }
26         else {
27             //分析出整数部分，小数位和小数部分；
28             //ex: v = 1.070m;
29
30             //整数部分太大，超过了int32的描述范围；
31             //当value == Int32.MinValue时，其变为正数后，比Int32.MaxValue大了1，超过范围，所以后面一个判断使用<=
32             if (value > Int32.MaxValue || value <= Int32.MinValue) {
33                 goto FullWrite;
34             }
35
36             if (value < 0) {
37                 flag = 0x4 | 0x2;
38                 e = value * -1;
39                 a = decimal.ToInt32(e);
40                 d = e - a;
41             }
42             else {
43                 flag = 0x4;
44                 //取出整数部分 a = 1
45                 a = decimal.ToInt32(value);
46                 //得到小数部分 d = 0.070m;
47                 d = value - a;
48             }
49
50             c = 0;
51             if (d == 0m) {
52                 //没有小数部分
53                 stream.WriteByte((byte)flag);
54                 context.Write7BitEncodedInt(a);
55                 goto NextFor;
56             }
57
58             do {
59                 e = d * Powers10[c]; //ex: 0.7 , 7
60                 b = decimal.ToInt32(e);
61                 c++;
62
63                 if (b == e) {//整数部分和小数部分相等，则得到小数部分的整数描述
64                     flag = (c << 3 | flag);
65                     stream.WriteByte((byte)flag);
66                     context.Write7BitEncodedInt(a);
67                     context.Write7BitEncodedInt(b);
68                     goto NextFor;
69                 }
70             } while (c < 9);
71
72             //小数部分太多，也使用原始方法写入。
73         FullWrite:
74             stream.WriteByte((byte)1);
75             //这里使用了BinaryWriter来填充到流，此实现bw实例级缓存了byte[]并调用decimal内部的
76             //的 GetBytes 方法，这比调用GetBits不断创建数组减少了开销。
77             context.Writer.Write(value);
78
79         NextFor:
80             flag = 0;
81         }
82     }
83 }
```

 

下面是读取时的方法：

 

```text
1 private static decimal[] PowersDecimal = new decimal[] {
 2     0.1m,
 3     0.01m,
 4     0.001m,
 5     0.0001m,
 6     0.00001m,
 7     0.000001m,
 8     0.0000001m,
 9     0.00000001m,
10     0.000000001m
11 };
12
13 private static void ReadFromFullByte(SerializeContext context, decimal[] values, int length) {
14     var stream = context.Stream;
15     int flag;
16     int a, b, c;
17     decimal d;
18
19     for (int i = 0; i < length; i++) {
20         flag = stream.ReadByte();
21         if (flag != 0) {
22             if (flag == 1) {
23                 //较大的数，系统内部完整数据
24                 values[i] = context.Reader.ReadDecimal();
25             }
26             else {
27                 c = flag >> 3;
28                 a = context.Read7BitEncodedInt();
29
30                 if (c > 0) {
31                     b = context.Read7BitEncodedInt();
32                     //乘法比除法快，在测试用例中，除法时，总时间是1分04秒，而乘法时为50秒
33                     d = a + ((decimal)b * PowersDecimal[c - 1]);
34                 }
35                 else {
36                     d = a;
37                 }
38
39                 if ((flag & 0x2) == 0x2) {
40                     values[i] = d * -1; //负数
41                 }
42                 else {
43                     values[i] = d;
44                 }
45             }
46         }
47     }
48 }
```

 

 

 

## 更进一步的优化

我在以上的基础上，进一步设想，

1、  如果这一列所有的数字都是0呢？这在ERP中非常常见，因为为了满足各种需求，设计人员总是设计一堆的字段，而这些字段一般用户根本不填写，全是0；

2、  如果这一列数字全是正整数，那么我是不是可以全部不要那个标志位字节，就节省一半的大小了，这种情况其实也常见，例如数量这一列，零售单明细中都是1~5之间的正整数，除非是称重的商品。

基于上面的想法，我在整列的存储中，使用一个字节描述存储方式，注意是整列的开头而不是每个数字的开头。这第一个字节：

1、  如果是0，表示所有的数字是0；

2、  如果是1，表示所有的数字都是小于0xFF的正整数，后面都使用一个字节描述其整数部分；

3、  如果是2，表示所有的数字都是小于0xFF FF的正整数，后面都使用2个字节描述其整数部分；

4、  如果是3，表示所有的数字都是小于0xFF FF FF的正整数，后面都使用3个字节描述其整数部分；

5、  如果是大于3的数字，表示此列使用本文描述的动态压缩方法存储数据。

下面是一段简单分析数据的方法，其返回此标志位：

```text
1 private static byte GetFirstFlag(decimal[] values) {
 2     decimal maxValue = 0;
 3     foreach (var item in values) {
 4         if (decimal.Floor(item) == item) {
 5             if (item > maxValue) {
 6                 maxValue = item;
 7                 if (maxValue > 0xFFFFFF) {
 8                     return 4;//大于3个byte可描述范围 使用full的算法
 9                 }
10             }
11             else if (item < 0) {
12                 return 4;//负数 使用string的算法
13             }
14         }
15         else {
16             return 4;//存在有效小数 使用full的算法
17         }
18     }
19
20     if (maxValue > 0xFFFF) {
21         return 3;// 65535 < maxValue <= 16777215 3个byte
22     }
23     else if (maxValue > 0xFF) {
24         return 2;//255 < maxValue <= 65535 2个byte
25     }
26     else if (maxValue > 0) {//0 < maxValue <= 255 1个byte
27         return 1;
28     }
29     return 0;
30 }
```

 

## 总结

通过不断分析ERP中的常见数据，理顺其规律，我们就能设计出更加优化的特定压缩算法。

[完整的测试代码请点击这里下载。](https://files.cnblogs.com/tansm/DecimalEncoder.zip)
