---
layout: post
title: "在 .NET 2.0 中享受 .NET 3.0的特性"
date: 2007-03-15 08:00:00 +0800
---

> 原文：https://www.cnblogs.com/tansm/archive/2007/03/15/675464.html
众多的新闻不断的向我轰炸，.NET 3.0是如何如何的好，但本人一直做商业产品的开发，看看可以，叫我在开发的产品中使用这些beta的东西，我是万万不敢的，出了问题老板非要宰了我不可。
可是向来对新东西很感兴趣的我，难耐“语法糖”的诱惑，特别是初始化器、自动属性和扩展函数，可以参考下面的文章知道他们的意思。

[Orcas中C#语言的新特性：自动属性，对象初始化器，和集合初始化器](http://blog.joycode.com/scottgu/archive/2007/03/11/95954.aspx)

[C# 3.0语言新特性（语言规范）：2 扩展方法](http://news.csdn.net/news/newstopic/26/26740.shtml)

从资料上分析，这些特性都是对编译器的改进，理论上.NET下也可以使用这些特性，能不能使用新的 Visual Studio 9.0(Orcas)开发程序，但是还是使用.NET 2.0编译呢？马上就试试看。

下载[最新的开发工具](http://www.microsoft.com/downloads/details.aspx?familyid=281fcb3d-5e79-4126-b4c0-8db6332de26e&displaylang=en)后，首先建立一控制台程序，注意在新建对话框的右上角选择.NET 2.0。
![](/images/2007-03-15-Enjoy-NET-3-0-Features-in-NET-2-0-1.jpg)
首先试验的是初始化器和自动属性，建立下面的程序：

```csharp
using System;
using System.Collections.Generic;

namespace ConsoleApplication1
{
    static class Program
    {
        static void Main(string[] args)
        {
            Employee e = new Employee { Name = "Hello" };
            Console.WriteLine("OK!{0}", e.Name);

            Console.ReadLine();
        }
    }

    public class Employee
    {
        public string Name { get; set; }
    }
}
```

编译，拷贝到仅安装.NET 2.0的机器，OK，非常顺利，反编译程序，发现编译器将你偷懒的东西补充上而已。

那么扩展方法如何呢 ，编写下面的程序：

```csharp
using System;

using System.Collections.Generic;
using System.Text;

namespace ConsoleApplication1
{
    static class Program
    {
        static void Main(string[] args)
        {
            string a = "32";
            int b = a.ToInt32();
            Console.WriteLine("Good!{0}",b);

            Console.ReadLine();
        }

        public static int ToInt32(this string s)
        {
            return Int32.Parse(s);
        }
    }
}
```

编译，说没有System.Core，这个是.NET 3.0东西，可以吗？管他的，找到DLL直接引用，Run，OK，拷贝到仅安装 .NET 2.0的机器，运行，耶！也OK，删除一并拷贝的System.Core.DLL，晕，还是可以。

总结：
.NET 3.0的很多语法优化都是编译器的功能，与.NET 3.0没有太多关系。
