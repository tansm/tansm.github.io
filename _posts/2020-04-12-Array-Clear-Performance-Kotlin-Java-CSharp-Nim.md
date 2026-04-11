# 一个清除数组的方法在 Kotlin、Java、C#和Nim上的性能测试

> 原文：https://www.cnblogs.com/tansm/p/12684664.html

## 起因

我的一个项目使用 Kotlin 编写，他是一个多维数据库应用程序，所以会非常频繁的操作 int 数组，其中有段程序就需要进行 几亿次的数组清除动作，类似这样的代码：

```text
Arrays.fill(target, 0);
```

这个Arrays.fill其实就是jdk自带的一个实现，非常简陋，就是一个for循环填充数据。

所以我想改进他，将常见的数组长度编写成单个的实现，比如清除8个长度的方法如下：

```text
fun clear8(target: IntArray) {
    if(target.size < 8){
        throw IndexOutOfBoundsException()
    }
    target[0] = 0
    target[1] = 0
    target[2] = 0
    target[3] = 0
    target[4] = 0
    target[5] = 0
    target[6] = 0
    target[7] = 0
}
```

不要怀疑你的眼睛，这样的写法通常是有效的。好的编译器会优化我写的代码，当然，更好的编译器会优化一个简单数组的for循环，这是后话。

那我们就测试一下吧。

```text
import java.util.*
import kotlin.system.measureNanoTime

fun main() {
    test3()
}

private fun test3() {
    val size = 8
    val time2 = measureNanoTime {
        val target = IntArray(size)
        for (i in 0 until 10_0000_0000) {
            IntArrays.clear8(target)
        }
    }
    println("fill$size          $time2")

    val time1 = measureNanoTime {
        val target = IntArray(size)
        for (i in 0 until 10_0000_0000) {
            Arrays.fill(target, 0)
        }
    }
    println("Arrays.fill$size   $time1")
    println()
}

internal object IntArrays {
    fun clear8(target: IntArray) {
        if(target.size < 8){
            throw IndexOutOfBoundsException()
        }
        target[0] = 0
        target[1] = 0
        target[2] = 0
        target[3] = 0
        target[4] = 0
        target[5] = 0
        target[6] = 0
        target[7] = 0
    }
}
```

测试结果：

fill8                    55,408,200
Arrays.fill8    2,262,171,100

可以看出，使用展开的方式，比java自带的2.2秒，性能提高了40倍！！

## 与Java的性能对比

我感叹kotlin的编译器真的很强，但仔细一想，不对啊， Kotlin 就是基于 JVM 的，功劳应该是 java 的虚拟机运行时很厉害，所以如果这个程序如果转化为java直接编写是不是更快，至少性能一致吧。说干就干。

![Image](/images/2020-04-12-Array-Clear-Performance-Kotlin-Java-CSharp-Nim-1.gif)

![Image](/images/2020-04-12-Array-Clear-Performance-Kotlin-Java-CSharp-Nim-2.gif)

```text
//IntArrays.java
import java.util.Arrays;

final class IntArrays {
    static void clear8(int[] target) {
/*        if (target.length < 8){
            throw new IndexOutOfBoundsException();
        }*/
        target[0] = 0;
        target[1] = 0;
        target[2] = 0;
        target[3] = 0;
        target[4] = 0;
        target[5] = 0;
        target[6] = 0;
        target[7] = 0;
    }
}

// IntArraysDemoJava.java
import java.util.Arrays;

public final class IntArraysDemoJava {
    public static void main(String[] var0) {
        test1();
    }

    private static void test1() {
        long count = 1000000000;
        long start = System.nanoTime();
        final int[] target = new int[8];

        for(int i = 0; i < count; i++) {
            IntArrays.clear8(target);
        }
        long time2 = System.nanoTime() - start;
        System.out.println("fill8          " + time2);

        start = System.nanoTime();
        for(int i = 0; i < count; i++) {
            Arrays.fill(target, 0);
        }

        long time1 = System.nanoTime() - start;
        System.out.println("Arrays.fill8   " + time1);
        System.out.println();
    }
}
```

Java的实现
测试结果如下：

fill8                   2,018,500,800
Arrays.fill8        2,234,306,500

天啊，在java下这种优化几乎没有效果，java我没有找到什么 release编译参数的概念，最多只有debug  = false，我是在gradle中包含

```text
compileJava {
    options.debug = false
}
```

那么就是说，Kotlin生成的字节码要好于 Java生成的字节码？

```text
Java               Kotlin
ALOAD 0         ALOAD 1
ICONST_0        ICONST_0
ICONST_0        ICONST_0
IASTORE         ASTORE

ALOAD 0         ALOAD 1
ICONST_1        ICONST_1
ICONST_0        ICONST_0
IASTORE         IASTORE
```

字节码稍微不同，你要是问我为什么？  我母鸡啊。。。。。。

## 与C# 的对比

作为一个 .net 的死忠粉，这个时候就会想着是不是 c# 更快一些，更何况 .net core 3做了大量的性能优化，

```text
class Program {
   static void Main(string[] args) {
       Test3.test1();
   }
}

class Test3
{
    public static void test1()
    {
        long count = 1000000000;
        var watch = System.Diagnostics.Stopwatch.StartNew();
        int[] target = new int[8];

        for (int i = 0; i < count; i++)
        {
            Clear8(target);
        }
        watch.Stop();
        Console.WriteLine("fill8          " + watch.Elapsed);

        watch.Restart();
        for (int i = 0; i < count; i++)
        {
            Array.Clear(target, 0,8);
        }

        watch.Stop();
        Console.WriteLine("Array.Clear8   " + watch.Elapsed);
        Console.WriteLine();
    }

    static void Clear8(int[] target)
    {
        /* if (target.Length < 8)
        {
            throw new IndexOutOfRangeException();
        }*/
        target[0] = 0;
        target[1] = 0;
        target[2] = 0;
        target[3] = 0;
        target[4] = 0;
        target[5] = 0;
        target[6] = 0;
        target[7] = 0;
    }
}
```

测试成绩：

fill8                     00:00:02.7462676
Array.Clear8      00:00:08.4920514

和Java比起来还要慢，甚至系统自带的Array.clear更加慢，这怎么能让我忍，于是一通的 Span.Fill(0)，结果更不理想。

## 和Nim对比的性能

兴趣提起来了，那就使用C语言实现一个....... 没写出来，我笨......，那就使用 Rust 实现一个，还是没有实现出来，按照教程一步步写，还是没有搞定..........

最后折腾出来一个 Nim 环境，嗯，还是这个简单。

![Image](/images/2020-04-12-Array-Clear-Performance-Kotlin-Java-CSharp-Nim-3.gif)

![Image](/images/2020-04-12-Array-Clear-Performance-Kotlin-Java-CSharp-Nim-4.gif)

```text
import times, strutils

proc clear8*[int](target: var seq[int]) =
    target[0] = 0
    target[1] = 0
    target[2] = 0
    target[3] = 0
    target[4] = 0
    target[5] = 0
    target[6] = 0
    target[7] = 0

proc clear*[int](target: var seq[int]) =
    for i in 0..<target.len:
        target[i] = 0

proc test3() =
    const size = 8
    var start = epochTime()
    var target = newseq[int](size)
    for i in 0..<10_0000_0000:
        target.clear8()

    let elapsedStr = (epochTime() - start).formatFloat(format = ffDecimal, precision = 3)
    echo "fill8         ", elapsedStr

    start = epochTime()
    for i in 0..<10_0000_0000:
        target.clear()

    let elapsedStr2 = (epochTime() - start).formatFloat(format = ffDecimal, precision = 3)
    echo "Arrays.fill   ", elapsedStr2

test3()
```

Nim
测试成绩，注意要加 --release 参数。

fill8 3.499
Arrays.fill   5.825

失望，及其失望。

## 备注

所有测试是在我的台式机上进行的，配置如下:

AMD Ryzen 5 3600 6 Core 3.59 Ghz

8 GB RAM

Windows 10 64 专业版

所有测试都使用release编译。
