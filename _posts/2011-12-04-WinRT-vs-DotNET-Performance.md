# WinRT比.NET快了，还是Win8比Win7快

> 原文：https://www.cnblogs.com/tansm/archive/2011/12/04/2275652.html

我对WinRT的理解是，WinRT是吸收了.net设计经验，一套非托管的Win32 API替代品。对于.net程序员来说，当我们调用WINRT库时，理论上，要比调用.NET自带的库要快，是吗？我今天就自己验证一下。

我的想法是使用WinRT自带的List<T>和.net Framework自带的List<T>进行比较，创建一个List<int>实例，然后添加两个实例，循环1亿次。

在Windows 8 64位开发者预览版下，我创建了一个页面，然后执行下面的代码：

```text
private void ListAddTest(){    System.Diagnostics.Stopwatch watch = new System.Diagnostics.Stopwatch();    watch.Start();    for (int i = 0; i < 100000000; i++)    {        List<int> list = new List<int>();        list.Add(i);        list.Add(i++);     }     watch.Stop();     lstOutput.Items.Add("Time=" + watch.Elapsed.ToString());}
```

测试结果基本在3.1-3.2之间，比较稳定。

 

然后，我重新启动，切换到本机另外一个操作系统,Windows 7 32位中文版，编写了一个控制台程序，使用.net framework 4 client profile。代码和上面一样。

测试结果基本在5.2-5.3之间，但很不稳定，偶发性的会快到4.7。

至此，我认为WinRT比.net快了，对吗？

 

我将Windows 7编译的程序重新在Windows 8环境下运行，测试结果竟然稳定在2.7-2.9秒之间。大出我意料！！

 

好吧，我犯糊涂了，

是Windows 8下，.net 运行效率提高很多？

是Windows 8下，程序自动切换到.net 4.5？是4.5比4.0快多了？

其实,WinRT下我并没有找到List<T>实现，其Windows.Runtime的DLL下，只有接口没有实现，我是在System.Collection.dll下找到的，是不是意味着我测试的实际上是.net 4.5与.net 4.0的比较？

求解？？

 

备注：

测试硬件为：Intel i3 M350 2.27GHz ，2GB 内存。

软件为：一个是Windows 7 32位中文版，一个是Windows 8 64位英文版（开发者预览版）。

编译环境：控制台程序在VS 2010 .net 4编译，WinRT程序在VS 12 .net 4.5下编译。所以编译均为Release版本，直接运行无调试。
