# C# 性能优化之斤斤计较篇  二

> 原文：https://www.cnblogs.com/tansm/archive/2012/05/14/dotnet_profile2.html

接[上篇](http://www.cnblogs.com/tansm/archive/2012/05/13/dotnet_profile1.html)继续，本文的完整源代码也在上篇文章中。

## 枚举数组和普通枚举性能差异

有些人可能知道，.net在处理枚举时，对于数组有特别的优化，所以，当枚举的集合是一个数组时，性能会好些。例如下面的测试代码：

```text
1     class C1 {
 2
 3         public void Do1() {
 4             int[] array = { 1, 2, 3, 4 };
 5             for (int i = 0; i < int.MaxValue/100; i++) {
 6                 DoIt1(array);
 7             }
 8         }
 9
10         private void DoIt1<T>(IEnumerable<T> array) {
11             foreach (var item in array) {
12
13             }
14         }
15
16         public void Do2() {
17             int[] array = { 1, 2, 3, 4 };
18             for (int i = 0; i < int.MaxValue/100; i++) {
19                 DoIt2(array);
20             }
21         }
22
23         private void DoIt2(int[] array) {
24             foreach (var item in array) {
25
26             }
27         }
28     }
```

 

第23行的方法中，编译器提前已知是一个数组的枚举，所以会优化指令。那么，到底这种优化差距有多大呢？我需要试验一下。

![Image](/images/2012-05-14-CSharp-Performance-Optimization-Part-2-1.png)

第一个是Do1的结果，第二个是Do2的结果，显而易见，差距还是相当大的，为什么呢？从反编译的IL代码来看，第一个方法使用标准的GetEnumerator机制，需要创建实例，而且要调用Current和MoveNext两个方法，更何况，Array的GetValue实现实在不够快。而第二个方法使用了for的机制，无需创建实例，不断累加和一个判断语句即可，性能当然高了。

在Linq to Object中，其实是有这样考虑的代码的。例如：

```text
public static IEnumerable<TResult> Select<TSource, TResult>(this IEnumerable<TSource> source, Func<TSource, TResult> selector)
{
    if (source == null)
    {
        throw Error.ArgumentNull("source");
    }
    if (selector == null)
    {
        throw Error.ArgumentNull("selector");
    }
    if (source is Enumerable.Iterator<TSource>)
    {
        return ((Enumerable.Iterator<TSource>)source).Select<TResult>(selector);
    }
    if (source is TSource[])
    {
        return new Enumerable.WhereSelectArrayIterator<TSource, TResult>((TSource[])source, null, selector);
    }
    if (source is List<TSource>)
    {
        return new Enumerable.WhereSelectListIterator<TSource, TResult>((List<TSource>)source, null, selector);
    }
    return new Enumerable.WhereSelectEnumerableIterator<TSource, TResult>(source, null, selector);
}
```

 

 

## 创建类和结构的性能差异以及属性和字段的性能差异

下面我还要试验创建类实例和结构类型，其性能差异到底有多大？会不会.net的垃圾回收超级厉害，基本上差异不大呢？当然，我也顺手测试了访问属性和访问字段的差别。

```text
1     class C1 {
 2         public void Do1() {
 3             for (int i = 0; i < int.MaxValue/10; i++) {
 4                 var p = new PointClass() { X = 1, Y = 2 };
 5             }
 6         }
 7         public void Do2() {
 8             for (int i = 0; i < int.MaxValue/10 ; i++) {
 9                 var p = new PointClass2() { X = 1, Y = 2 };
10             }
11         }
12         public void Do3() {
13             for (int i = 0; i < int.MaxValue/10; i++) {
14                 var p = new Point() { X = 1, Y = 2 };
15             }
16         }
17         public void Do4() {
18             for (int i = 0; i < int.MaxValue / 10; i++) {
19                 var p = new Point() { XP = 1, YP = 2 };
20             }
21         }
22     }
23
24
25     class PointClass {
26         public int X { get; set; }
27         public int Y { get; set; }
28     }
29
30     class PointClass2 {
31         public int X;
32         public int Y;
33     }
34
35     struct Point {
36         public int X;
37         public int Y;
38
39         public int XP { get { return X; } set { X = value; } }
40         public int YP { get { return Y; } set { Y = value; } }
41     }
```

 

测试结果如下：

![Image](/images/2012-05-14-CSharp-Performance-Optimization-Part-2-2.png)

实验结果表明：在计算密集型的程序中，结构的创建仍然比类要高效的多，另外，属性和字段的访问其性能基本相当。
