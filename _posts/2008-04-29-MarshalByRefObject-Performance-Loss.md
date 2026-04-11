---
title: "MarshalByRefObject 的性能损失"
date: 2008-04-29 00:00:00 +0800
---

# MarshalByRefObject 的性能损失

以前看过文章说 MarshalByRefObject 会造成性能的损失，我比较相信自己，所以亲自测试了一下，下面是代码：

```text
测试代码

using System;

using System.Diagnostics;



namespace ConsoleApplication1 {

    class Program {

        static void Main(string[] args) {

            Stopwatch watch = new Stopwatch();

            watch.Start();

            B b = new B();

            for (int i = 0; i < 100000000; i++) {

                b.Add(3, 5);

            }

            watch.Stop();

            Console.WriteLine("B 花费时间：" + watch.ElapsedMilliseconds.ToString());

            watch.Reset();



            watch.Start();

            A a = new A();

            for (int i = 0; i < 100000000; i++) {

                a.Add(3, 5);

            }

            watch.Stop();

            Console.WriteLine("A MarshalByRefObject 花费时间：" + watch.ElapsedMilliseconds.ToString());

            watch.Reset();



            //=======================

            watch.Start();

            a = new A();

            for (int i = 0; i < 100000000; i++) {

                a.Add(3, 5);

            }

            watch.Stop();

            Console.WriteLine("A MarshalByRefObject 花费时间：" + watch.ElapsedMilliseconds.ToString());

            watch.Reset();



            watch.Start();

            b = new B();

            for (int i = 0; i < 100000000; i++) {

                b.Add(3, 5);

            }

            watch.Stop();

            Console.WriteLine("B 花费时间：" + watch.ElapsedMilliseconds.ToString());

            watch.Reset();

            Console.Read();

        }

    }



    class A : MarshalByRefObject {

        public int Add(int a, int b) {

            return a + b;

        }

    }



    class B {

        public int Add(int a, int b) {

            return a + b;

        }

    }

}
```

测试的结果是：

B 花费时间：55

A MarshalByRefObject 花费时间：957

A MarshalByRefObject 花费时间：972

B 花费时间：56

总结：像这样在本地环境下，性能仍然损失了近 17.4 倍。当然，此 17 倍不能简单地理解为你的应用就慢了 17 倍，这里仅表示发起调用损失了 17 倍。

**注意：**

执行测试程序时，首先选择 Release，然后选择项目的属性 => Build（编译）=> 高级 => 调试信息，设置为 none。然后选择：调试 => 不调试运行。或找到 exe 直接双击运行。
