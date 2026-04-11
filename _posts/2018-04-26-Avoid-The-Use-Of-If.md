# 数学运算比IF要快

> 原文：https://www.cnblogs.com/tansm/p/AvoidTheUseOfIf.html

## 问题

虽然很早就知道，CPU在处理 if 这样的判断语句时，使用了预测的技术，所以如果条件总是一个结果，效率就很好。反过来说，如果你使用数学运算避免 if 判断，那么就意味着性能一定比 if 要好。

## 方案1

今天正好有个函数遇到这个问题，所以我就正好测试以下。

待测试的方法是获取一个int32的数据中，有多少个位是 1，我的方案是将 一个int32拆开成 4个字节，然后一一判断。下面是使用 if 判断的方案 （那个  ? : 三元运算符就是 if 语句）。

```text
1 static int getInt32TrueCount(int value)
 2 {
 3     if (value == 0)
 4     {
 5         return 0;
 6     }
 7
 8     return getByteTrueCount(value & 0xFF) +
 9         getByteTrueCount((value >> 8) & 0xFF) +
10         getByteTrueCount((value >> 16) & 0xFF) +
11         getByteTrueCount((value >> 24) & 0xFF);
12 }
13
14 static int getByteTrueCount(int value)
15 {
16     if (value == 0)
17     {
18         return 0;
19     }
20
21     int a = (value & 0x1) == 0 ? 0 : 1;
22     int b = (value & 0x2) == 0 ? 0 : 1;
23     int c = (value & 0x4) == 0 ? 0 : 1;
24     int d = (value & 0x8) == 0 ? 0 : 1;
25
26     int e = (value & 0x10) == 0 ? 0 : 1;
27     int f = (value & 0x20) == 0 ? 0 : 1;
28     int g = (value & 0x40) == 0 ? 0 : 1;
29     int h = (value & 0x80) == 0 ? 0 : 1;
30
31     return a + b + c + d + e + f + g + h;
32 }
```

可以看到， 每运算一个位都有一个 if 判断，而要命的是这个 if 判断的结果是不稳定的，随机性极大。我写了一个计时程序，在我计算机中需要 12秒。（i5 6500 Release .net core 2.0 )

```text
1 static void GetBitCountTest()
 2 {
 3     var wathch = Stopwatch.StartNew();
 4
 5     var rand = new Random();
 6
 7     for (int i = 0; i < 10000_0000; i++)
 8     {
 9         int value = rand.Next();
10         int p = getInt32TrueCount(value);
11     }
12
13     wathch.Stop();
14     Console.WriteLine("GetBitCount 耗时：" + wathch.Elapsed.ToString());
15
16 }
```

 

方案2

第二种方法就是将 if 判断改为数学运算，方法是将 and 运算后的位 移动到 0位，这样就是 0 或 1 了。

```text
1 static int getInt32TrueCount2(int value) {
 2     if (value == 0) {
 3         return 0;
 4     }
 5
 6     return getByteTrueCount2(value & 0xFF) +
 7         getByteTrueCount2((value >> 8) & 0xFF) +
 8         getByteTrueCount2((value >> 16) & 0xFF) +
 9         getByteTrueCount2((value >> 24) & 0xFF);
10 }
11
12 static int getByteTrueCount2(int value) {
13     return (value & 0x1) +
14             ((value & 0x2) >> 1) +
15             ((value & 0x4) >> 2) +
16             ((value & 0x8) >> 3) +
17
18             ((value & 0x10) >> 4) +
19             ((value & 0x20) >> 5) +
20             ((value & 0x40) >> 6) +
21             ((value & 0x80) >> 7);
22 }
```

再次运行测试用例，执行时间提高到 2 秒！提高了6倍。

 

## 总结：

高性能计算时，避免使用 分支 指令，尽量使用数学运算符。
