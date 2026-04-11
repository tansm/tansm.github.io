---
title: "Struct 和 Class 性能有差异吗？自己测试"
date: 2010-03-15 00:00:00 +0800
---

一直以来，我总是听到关于在字段较少时，使用结构将获得更佳的性能，我对此一直深信不已，今天因为需要写一个性能要求较高的程序，所以特地写一段测试程序来验证是不是真的正确。
    我建立了1个结构以及2个类：

    struct MyStruct
    {
        public int a;
        public string b;
    }
    class MyClass
    {
        public int a;
        public string b;
    }
    sealed class MyClassReadonly
    {
        public MyClassReadonly(int x, string y)
        {
            a = x;
            b = y;
        }
        public readonly int a;
        public readonly string b;
    }

     第3个类和第2个类的区别是：使用了readonly访问符，他也是被告知有益于提高性能。
     测试程序比较简单，测试对象的创建、赋值和访问。
     计算机环境是：Windows 7 旗舰版，Intel E5200 2.5G，2GB RAM。.net 4.0 Release编译。

    class Program
    {
        static void Main(string[] args)
        {
            int count = 30000000;
            System.Diagnostics.Stopwatch watch = new System.Diagnostics.Stopwatch();
            int x; string y;
            watch.Start();
            MyStruct a;
            for (int i = 0; i < count; i++)
            {
                a = new MyStruct();
                a.a = i;
                a.b = i.ToString();
                x = a.a;
                y = a.b;
            }
            watch.Stop();
            Console.WriteLine("Struct:{0}", watch.Elapsed);
            watch.Reset();
            watch.Start();
            MyClass b;
            for (int i = 0; i < count; i++)
            {
                b = new MyClass();
                b.a = i;
                b.b = i.ToString();
                x = b.a;
                y = b.b;
            }
            watch.Stop();
            Console.WriteLine("Class:{0}", watch.Elapsed);
            watch.Reset();
            watch.Start();
            MyClassReadonly c;
            for (int i = 0; i < count; i++)
            {
                c = new MyClassReadonly(i,i.ToString());
                x = c.a;
                y = c.b;
            }
            watch.Stop();
            Console.WriteLine("Readonly Class:{0}", watch.Elapsed);
            Console.Read();
        }
    }

     最终的测试结果如下：
Struct:00:00:04.7962253
Class:00:00:04.9951920
Readonly Class:00:00:05.1693143

     总结：
     1、结构的确比类快，但性能提高的微乎其微；
     2、多一个构造比默认的构造要慢，但影响微乎其微；
     3、readonly关键字对性能的提高微乎其微；
     也就是说，觉得哪个好用就用哪个吧，性能差不多。

	posted on
2010-03-15 15:27
[编写人生](https://www.cnblogs.com/tansm)
阅读(732)
评论(5)

[收藏](javascript:void(0))
[举报](https://report.cnblogs.com?targetLink=https%3A%2F%2Fwww.cnblogs.com%2Ftansm%2Farchive%2F2010%2F03%2F15%2F1686270.html&targetId=1686270&targetType=0)
