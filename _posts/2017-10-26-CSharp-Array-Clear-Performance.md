# 清除数组数据的正确姿势

> 原文：https://www.cnblogs.com/tansm/p/7739212.html

因为某段程序的需要，我需要将一个long数组，不断地填充数据，然后用完了之后又要清空里面的数据，以便再次填充。由于调用及其频繁，所以我很在意清除数据的性能。

## 测试代码

以下程序都是基于下面的测试代码完成：

```text
using System;
using System.Diagnostics;
using System.Numerics;

namespace FillTest
{
    class Program
    {
        static void Main(string[] args)
        {
            var array = new long[2515];
            var Empty = new long[2515];

            Do(() =>
            {
                Fill(array, 0L);
            }
            , "Fill");

            Console.ReadLine();
        }

        static void Fill(long[] array,long value)
        {
            for (int i = 0; i < array.Length; i++)
            {
                array[i] = value;
            }
        }

        static void Do(Action action,string name)
        {
            var wathch = Stopwatch.StartNew();
            for (int i = 0; i < 1000000; i++)
            {
                action();
            }

            wathch.Stop();
            Console.WriteLine(name + " 耗时：" + wathch.Elapsed.ToString());
        }
    }
}
```

通常的，最简单的办法就像上面的代码那样，来个循环就可以了。

在我的机器中（i5 6500@3.2GHz 、16GB、 Release、 .net core 2、 windows 10），Fill版本消耗1.3秒。

## 双指令

我不确认此代码执行时是否使用了SIMD指令，所以我又编写了一个版本：

```text
static void Fill2(long[] array,long value)
        {
            var i = 0;
            var end = array.Length - 1;
            while (i < end)
            {
                array[i] = value;
                array[i + 1] = value;
                i += 2;
            }

            if((array.Length % 2) == 1){
                array[array.Length - 1] = value;
            }
        }
```

我居然发现，仅仅消耗了1.0秒，难道.net默认不会优化这个代码？

## Array.Fill

有人会问，你为什么不调用Array自带的Fill方法呢？我尝试调用了，遗憾的是，一样是1.3秒，和我的第一个版本系统。看样子写核心代码的人偷懒了。哈哈

## CopyTo

网上其实能够查到一些讨巧的手法，例如使用一个静态的数组，内部都是0，当需要填充某个数据时，将这个静态数组复制到你的数组一样起到赋值的作用。例如：

```text
Empty.CopyTo(array, 0);
```

这个版本竟然直接达到 0.56秒，几乎翻两倍，的确是个好办法。

## SIMD

在我认为CopyTo是最快的方法时，我突然感觉我的第二个实现，是不是没有真正用到SIMD，因为我的印象中，SIMD是非常非常快的，应该不止上升那么一点点。

所以我搬出了Vector对象，我是这么干的。

```text
static void Fill3(long[] array,long value)
        {
            Vector<long> v = new Vector<long>(value);
            var i = 0;
            var end = array.Length - 3;

            while (i < end)
            {
                v.CopyTo(array, i);
                i += 4;
            }

            if ((array.Length % 4) == 3)
            {
                array[array.Length - 3] = value;
                array[array.Length - 2] = value;
                array[array.Length - 1] = value;
            }else if ((array.Length % 4) == 2)
            {
                array[array.Length - 2] = value;
                array[array.Length - 1] = value;
            }
            else if ((array.Length % 4) == 1)
            {
                array[array.Length - 1] = value;
            }
        }
```

速度提高到 0.36秒。

所以性能无界限，有没有更快的办法呢？

## 后续

一些其他的尝试，但都不理想。

```text
static unsafe void Fill4(long[] array, long value)
        {
            fixed (Int64* destinationBase = array)
            {
                for (int g = 0; g < array.Length; g++)
                {
                    destinationBase[g] = value;
                }
            }
        }//1.2秒

        static void Fill5(long[] array, long value)
        {
            Vector<long> v = new Vector<long>(value);
            var i = 0;
            var end = array.Length - 3;

            while (i < end)
            {
                array[i + 3] = array[i + 2] =array[i + 1] = array[i] = value;
                //array[i+1] = value;
                //array[i+2] = value;
                //array[i+3] = value;

                i += 4;
            }

            if ((array.Length % 4) == 3)
            {
                array[array.Length - 3] = value;
                array[array.Length - 2] = value;
                array[array.Length - 1] = value;
            }
            else if ((array.Length % 4) == 2)
            {
                array[array.Length - 2] = value;
                array[array.Length - 1] = value;
            }
            else if ((array.Length % 4) == 1)
            {
                array[array.Length - 1] = value;
            }
        } //0.8秒
```
