# Kotlin 使用类似C# 的yield功能

> 原文：https://www.cnblogs.com/tansm/p/9211244.html

用过c#的可能对 yield 关键字爱不释手，那么在像我这种被迫上java贼船的人，就想找到类似的功能。

我使用的是kotlin，下面的方法演示了产生一个序列的功能。

```text
val fibonacciSeq = buildSequence {
    var a = 0
    var b = 1

    yield(1)

    while (true) {
        yield(a + b)

        val tmp = a + b
        a = b
        b = tmp
    }
}

fun main(args: Array<String>){
    fibonacciSeq.take(50).forEach {
        println(it)
    }
}
```

程序将执行50次然后退出。

 

## 实现枚举器

上面的代码使用的是序列功能，并非是常见的枚举器，下面的代码更像c#的枚举器。

```text
class MyList : Iterable<Int>{
    private val list = arrayOf(1,2,3,4,5,6)

    override fun iterator(): Iterator<Int> {
        return buildIterator {
            val size = list.size
            for(i in 0 until size) {
                yield(list[i] + i)
            }
        }
    }
}

fun main(args: Array<String>){
val list = MyList()
    for (p in list) {
        println(p)
    }
}
```

 

## 实现类似Linq

我们在使用Linq时可以将序列不断的变换，Kotlin也可以很方便的处理。

```text
//将输入的数据 + 1
fun Do1(seq : Sequence<Int>) : Sequence<Int>{
    return buildSequence {
        for (i in seq){
            yield(i + 1)
        }
    }
}

//变换成字符串
fun Do2(seq : Sequence<Int>) : Sequence<String>{
    return buildSequence {
        for (i in seq){
            yield("hello $i ")
        }
    }
}

//将多个数据聚合到一个数据
fun Do3(seq : Sequence<String>) : Sequence<String>{
    return buildSequence{
        var result = ""
        var count = 0

        for (i in seq){
            result += i
            count++
            if(count == 3){
                yield(result)
                result = ""
                count = 0
            }
        }

        if(count > 0){
            yield(result)
        }
    }
}

fun main(args: Array<String>){
    val data = arrayOf(1,2,3,4,5,6,7,8)
    val result = Do3(Do2(Do1(data.asSequence())))
    for (p in result){
        println(p)
    }
}
```
