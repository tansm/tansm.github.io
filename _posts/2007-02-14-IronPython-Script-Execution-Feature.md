---
layout: post
title: "牛刀小试：IronPython的脚本执行功能"
date: 2007-02-14 08:00:00 +0800
---

> 原文：https://www.cnblogs.com/tansm/archive/2007/02/14/650180.html
昨天和今天一直在捉摸这个IronPython（铁蟒蛇？），早就听说这个东西，但真的精力有限，所以一直没有学习。最近因为要考虑为我们的应用程序添加脚本功能，所以开始简单试用了一下。
语法吗，没有什么好说的，网上的资料满天飞，本身也自带了非常好的教程。
我比较喜欢的功能有：
— 很好的与.NET和Com互操作；
— 可以生成静态的DLL到文件中，创建的文件包含PDB，因此你可以做断点、察看变量，晕死，强；
— 使用.NET可以非常方便的使用他的引擎，网上的很多资料都旧了些，所以我自己写了一个，下面的演示创建的代码与你自己的程序产生了交互。

```csharp
using System;
using IronPython.Hosting;

namespace ConsoleApplication1 {
    class Program {
        static void Main(string[] args) {
            A a = new A();
            string code = @"A.Amount = 3";

            PythonEngine engine = new PythonEngine();
            EngineModule dbafModule = engine.CreateModule("DBAF", true);
            dbafModule.Globals.Add("A", a);
            engine.Execute(code, dbafModule);

            Console.WriteLine(a.Amount);
            Console.ReadLine();
        }
    }

    public class A {

        private int _amount;

        public int Amount {
            get { return _amount; }
            set { _amount = value; }
        }
    }
}
```
