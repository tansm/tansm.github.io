# 发现.net 4关于编译匿名函数的一个BUG

> 原文：https://www.cnblogs.com/tansm/archive/2011/05/20/2051684.html

测试环境：

Windows 7 中文版

Visual Studio 2010 英文版 10.0.31118.1 SPRel

.net 4.0.30319 sp1Rel 英文版

BUG描述：

在一个循环中，如果循环内部创建了一个实例级的匿名委托，并让此委托持有一个实例，最后将此委托放入数组。

那么运行时的效果是：数组中的所有委托均指向同一个实例。

论证：

建立控制台应用程序，.net版本为4.0。

```text
1 using System;  2  using System.ComponentModel;  3  4  namespace LambdaTest  5 {  6     class Program  7     {  8         static void Main(string[] args)  9         { 10             var temp = GetConverter(); 11 12         } 13 14         public static Func<object, object>[] GetConverter() 15         { 16             Func<object, object>[] converters = new Func<object, object>[5]; 17             Property p; 18             Func<object, object> func; 19 20             for (int i = 0; i < 5; i++) 21             { 22                 p = new Property() { Name = "P" + i.ToString() }; 23                 func = (v) => p.Converter.ConvertTo(v, typeof(Int64)); 24                 converters[i] = func; 25             } 26 27             return converters; 28         } 29     } 30 31     class Property 32     { 33         public Property() 34         { 35             //test 36              this.Converter = new Int64Converter(); 37         } 38         public string Name; 39 40         public virtual TypeConverter Converter { get; private set; } 41     } 42 }
```

在第25行设置断点，会发现，当i = 0时，其委托的目标对象的Name = "P0"，但当i=1时，你会发现数组中，位置0的对象其目标属性Name 变更为Name = "P1"，未达到我们预想的，每个委托挂接不同的实例。

分析：

通过反编译工具查看，（调整Reflector的选项，设置优化为 none）。

```text
1 public static unsafe Func<object, object>[] GetConverter()  2 {  3     Func<object, object>[] converters;  4     Func<object, object> func;  5     int i;  6     Property <>g__initLocal0;  7     Func<object, object> CS$<>9__CachedAnonymousMethodDelegate2;  8     <>c__DisplayClass3 CS$<>8__locals4;  9     CS$<>9__CachedAnonymousMethodDelegate2 = null; 10     CS$<>8__locals4 = new <>c__DisplayClass3(); 11     converters = new Func<object, object>[5]; 12     i = 0; 13     goto Label_0058; 14 Label_0015: 15     <>g__initLocal0 = new Property(); 16     <>g__initLocal0.Name = "P" + &i.ToString(); 17     CS$<>8__locals4.p = <>g__initLocal0; 18     if (CS$<>9__CachedAnonymousMethodDelegate2 != null) 19     { 20         goto Label_004D; 21     } 22     CS$<>9__CachedAnonymousMethodDelegate2 = new Func<object, object>(CS$<>8__locals4.<GetConverter>b__1); 23 Label_004D: 24     func = CS$<>9__CachedAnonymousMethodDelegate2; 25     converters[i] = func; 26     i += 1; 27 Label_0058: 28     if (i < 5) 29     { 30         goto Label_0015; 31     } 32     return converters; 33 }
```

我发现编译器错误的进行了优化，仅在第一次创建委托实例（第18行），而此委托持有的实例p（第17行），在循环中不断改变。

我记忆中，C#的编译器在处理静态方法时，有时候的确会作此优化，之前也看过其他的反编译，当时还觉得编译器真聪明。根据此观点，我将函数改为实例级的，还是不行。

继续，通过将程序23行修改，改为调用一个方法获取委托：

```text
func = GetFunc(p)
```

其GetFunc(p)定义如下：

```text
1 public static Func<object,object> GetFunc(Property p)2 {3     return (v) => p.Converter.ConvertTo(v,typeof(Int64));4 }
```

通过此改动，编译器编译的结果正确。
