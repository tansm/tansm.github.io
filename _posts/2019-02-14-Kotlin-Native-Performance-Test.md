# 简单测试 Kotlin native  性能

> 原文：https://www.cnblogs.com/tansm/p/KotlinNativeTest.html

## 准备

一直使用kotlin JVM平台开发服务器的应用，最近想试试看 Kotlin native的性能。

我使用的是 kotlin native 1.3.21，要使用他非常的简单，下载最新的 IDEA ，我下载的是 IntelliJ IDEA 2018.3.4 (Community Edition)，然后新建项目时选择 “Kotlin/Native”，非常的简单了。

测试环境如下：

Windows 10 64 位

Intel Core i5-6500 @3.2GHz   4 Core

16GB RAM

## 测试代码

这个项目还在初期，所以对应的库一定还不成熟，所以，我尽力避免使用库，而且不同的库实现不同和使用不当，都可能造成测试不准确。

所以我测试简单的循环，int的位操作，这些指令都是对编译器的考验，下面的测试代码就是检测一个int32值，包含几个有效的 1 位。

```text
package sample

//import kotlin.random.Random
import kotlin.system.measureNanoTime

fun main() {
    runIt()
}

private fun runIt(){
    var sum = 0
    val time = measureNanoTime{
        //val ran = Random.Default
        for (i in 0 until 1_0000_0000){
            //val v = ran.nextInt()
            sum += getInt32TrueCount(i)
        }
    }
    // 292 056 900
    println("共耗时：$time ns, result: $sum")
}

private fun getInt32TrueCount(value: Int):Int {
    if (value == 0) {
        return 0
    }

    return getByteTrueCount(value and 0xFF) +
            getByteTrueCount((value shr 8) and 0xFF) +
            getByteTrueCount((value shr 16) and 0xFF) +
            getByteTrueCount((value shr 24) and 0xFF)
}

private fun getByteTrueCount(value: Int) : Int{
    if(value== 0){
        return 0
    }

    val a = (value and 0x1)
    val b = ((value and 0x2)  shr 1)
    val c = ((value and 0x4)  shr 2)
    val d = ((value and 0x8)  shr 3)
    val e = ((value and 0x10) shr 4)
    val f = ((value and 0x20) shr 5)
    val g = ((value and 0x40) shr 6)
    val h = ((value and 0x80) shr 7)

    return a + b + c + d + e + f + g + h
}
```

## 测试结果

Kotlin有个非常大的好处，常见的库都可以在 jvm 平台和 native 平台通用，所以上面的代码可以直接复制到 Kotlin 的jvm环境下执行。

在 Gradle 面板中，找到 Tasks -> run -> runMainReleaseExecutableMingw，就可以运行程序。

耗时如下：

Kotlin Native :     292 056 900 ns

Kotlin Jvm    ：1 220 617 300 ns

可以明显看见，native是jvm的 4被性能，我在怀疑是不是 native 的LLVM编译器 实现了并行，不然怎么差不多4倍呢？

 坑

你可能注意到，我注释了随机数产生的函数调用，这是因为我发现 native 平台下，默认的随机数产生非常的慢，远远慢于 Jvm 平台。所以库可能不太成熟。

 

## SIMD

LLVM平台的最大亮点是性能的优化，比如 SIMD，所以我尝试修改程序，看看是否能启用SIMD，所以我修改了函数，新代码如下：

```text
private val m1 = intArrayOf(0x1,0x2,0x4,0x8,0x10,0x20,0x40,0x80)

private fun getByteTrueCount(value : Int) : Int{
    if (value == 0) {
        return 0
    }

    var sum = 0
    for (i in 0 until m1.size){
        sum += (value and m1[i] shr i)
    }

    return sum
}
```

然而，悲剧发生了，执行时间如下：

Kotlin Native :   80 610 886 500 ns

Kotlin Jvm    ：   1 297 672 800 ns

最终，native平台花了整整80多秒，你没有看错， native慢了很多很多，而JVM平台似乎能聪明的实现了SIMD优化(我猜的）。

至于为什么，我无法知道，

所以，还是比较多的坑。

 

## 后记

我提了一个 Issue 给了kotlin native，参见： https://github.com/JetBrains/kotlin-native/issues/2660

他们建议不要使用全局的数组，我修改了，效果不明显。

另外，他们还建议使用 GCC的内置函数 __builtin_popcount

至于为什么 native 比 Jvm 平台这段代码要慢的原因，他们的意见是现在还是 beta 哪，[不要做性能测试](https://github.com/JetBrains/kotlin-native/blob/master/RELEASE_NOTES.md#performance)。

虽然有一些问题，但是我任然非常期待 Kotlin native，这样学会一个 kotlin ，就所有平台通杀了。
