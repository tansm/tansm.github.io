## C# 泛型编译特性对性能的影响
C#作为一种强类型语言，具有丰富的泛型支持，允许开发者编写可以应对不同数据类型的通用代码。然而，在泛型编译时，针对结构和类作为泛型参数时，会对性能产生不同的影响。

### 泛型编译行为
在C#中，泛型编译行为取决于泛型参数的类型。具体而言，当泛型参数是结构（Struct）时，编译器会针对每个具体的结构类型生成特定的实现。而当泛型参数是类（Class）时，编译器则可能生成更通用的实现。

### 结构 vs 类
#### 结构（Struct）
结构是值类型，它们存储在栈上，具有较小的内存开销。当泛型参数是结构时，编译器会针对每个具体的结构类型生成专门的实现，这可能导致更高的性能。因为每个结构类型都有自己的实现，避免了装箱和拆箱的开销，同时优化了内存分配和访问。

#### 类（Class）
类是引用类型，存储在堆上，需要通过引用进行访问。当泛型参数是类时，编译器可能生成更通用的实现。这可能导致较低的性能，因为通用实现需要进行动态调度和引用类型的操作，增加了一些开销。

### 测试性能差异
针对不同的泛型参数进行性能测试是一种有效的方法，以观察结构和类对泛型编译特性的影响。在测试中，可能会发现对结构类型的泛型参数，其性能可能更高，而对类类型的泛型参数，其性能可能略低。

``` cs
using System.Diagnostics;

namespace ConsoleApp1 {
    internal interface IValueGetter {
        int GetValue(int index);
    }

    internal class MyTestClass<T> where T : IValueGetter {
        private readonly T _valueGetter;

        public MyTestClass(T valueGetter) {
            _valueGetter = valueGetter;
        }

        public void Run() {
            long r = 0L;
            for (int i = 0; i < int.MaxValue; i++) {
                r += _valueGetter.GetValue(i);
            }
        }
    }

    internal struct StructValueGetter : IValueGetter {
        public readonly int GetValue(int index) {
            return index + 3;
        }
    }

    internal struct StructValueGetter2(int someField) : IValueGetter {
        public readonly int GetValue(int index) {
            return index + 5;
        }
    }

    internal class ClassValueGetter1 : IValueGetter {
        public int GetValue(int index) {
            return index + 5;
        }
    }

    internal class ClassValueGetter2 : IValueGetter {
        public int GetValue(int index) {
            return index + 7;
        }
    }

    internal static class Demo2 {
        public static  void Run() {
            var t1 = new MyTestClass<StructValueGetter>(new StructValueGetter());
            RunDemo("StructValueGetter ", t1.Run);
            var t2 = new MyTestClass<ClassValueGetter1>(new ClassValueGetter1());
            RunDemo("ClassValueGetter1 ", t2.Run);
            var t3 = new MyTestClass<ClassValueGetter2>(new ClassValueGetter2());
            RunDemo("ClassValueGetter2 ", t3.Run);
            var t4 = new MyTestClass<IValueGetter>(new ClassValueGetter1());
            RunDemo("IValueGetter-1    ", t4.Run);


            var t5 = new MyTestClass<ClassValueGetter1>(new ClassValueGetter1());
            RunDemo("ClassValueGetter1 ", t5.Run);
            var t6 = new MyTestClass<StructValueGetter2>(new StructValueGetter2());
            RunDemo("StructValueGetter2", t6.Run);
            var t7 = new MyTestClass<IValueGetter>(new ClassValueGetter2());
            RunDemo("IValueGetter-2    ", t7.Run);
            var t8 = new MyTestClass<IValueGetter>(new StructValueGetter());
            RunDemo("IValueGetter-3    ", t8.Run);

            var t9 = Activator.CreateInstance(typeof(MyTestClass<>).MakeGenericType(typeof(StructValueGetter)), new StructValueGetter());
            Action action9 = (Action)Delegate.CreateDelegate(typeof(Action), t9, t9.GetType().GetMethod("Run"));
            RunDemo("Dynamic-Struct    ", action9);

        }

        static void RunDemo(string caption, Action action) {
            var stopWatch = Stopwatch.StartNew();
            action();
            stopWatch.Stop();
            Console.WriteLine($"{caption} time = {stopWatch.Elapsed}");
        }
    }
}

Demo2.Run();

```

在.net 8.0 Release 编译执行的参考结果如下：
``` text
StructValueGetter  time = 00:00:00.6920186
ClassValueGetter1  time = 00:00:01.1887137
ClassValueGetter2  time = 00:00:05.2889692
IValueGetter-1     time = 00:00:01.1652195
ClassValueGetter1  time = 00:00:01.1625259
StructValueGetter2 time = 00:00:00.6488674
IValueGetter-2     time = 00:00:05.2114724
IValueGetter-3     time = 00:00:07.1394676
Dynamic-Struct     time = 00:00:00.6491220
```

### 结论
泛型编译特性对性能有所影响，我们发现：
- 泛型参数是 Struct 比 class 的性能要好，大约有两倍的差异；
- 泛型参数如果存在多个 Struct 可能时，性能没有影响，但如果泛型参数存在多个 class 可能时，性能急剧下降5倍之多；
- 泛型参数如果是接口形式，无论实际填充的结构还是类，其最终的执行性能一定是很慢的；
- 使用反射（例如：MakeGenericType）构建出的泛型实例，其实际运行性能并不受影响，非常适合高度定制的运行时类型构建，这一点非常重要，例如你可以在运行时检测实际情况，构建出不同的比较器对象，虽然构建的工厂方法返回的是接口，但你可以使用反射的方式动态传入字典的比较器参数（实际上c#的 `Dictionary<TKey, TValue>` 这点设计是失败的，他的`comparer`不是一个泛型参数，而是接口）；

综上所述，了解C#泛型编译特性对性能的影响是编写高性能代码的重要一部分，合理使用对于关键性代码性能至关重要。