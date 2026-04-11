---
layout: post
title: "使用IronPython分析Lambda表达式"
date: 2007-10-17 08:00:00 +0800
---
在我们的项目中，要使用到自定义公式功能，我们利用了IronPython的Lambda功能，可以方便的计算值，但是我们发现，如果表达式涉及的属性发生改变时，公式必须重新计算，怎样析表达式知道到底访问了哪些属性呢？

仔细研究发现IronPython提供了这样的功能：

```
SystemState state = new SystemState();

            CompilerContext context = new CompilerContext();

            Parser p = Parser.FromString(state, context, "ActiveObject.Contact.Address + 'ds'");

            IronPython.Compiler.Ast.Expression ex = p.ParseTestListAsExpression();
```

上面的代码分析了表达式：ActiveObject.Contact.Address + 'ds'， 这里用了Parser类可以分析出表达式，使用VS对象查看器，可以理解结果：

![](/images/2007-10-17-IronPython-Lambda-Expressions-1.jpg)

问题还没有完全解决，我想结果有了，我总不能自己递归所有结构吧？哇，怎么也有20多种类型，在看看，嗯，还有这个：

```csharp
class Program {

        static void Main(string[] args) {

            SystemState state = new SystemState();

            CompilerContext context = new CompilerContext();

            Parser p = Parser.FromString(state, context, "ActiveObject.Contact.Address + 'ds'");

            IronPython.Compiler.Ast.Expression ex = p.ParseTestListAsExpression();

            MyWalker w = new MyWalker();

            ex.Walk(w);

        }

    }

    class MyWalker : AstWalker {

        public override bool Walk(FieldExpression node) {

            Console.WriteLine("Walk:{0},{1}", node.Name, node.Target);

            return base.Walk(node);

        }

    }
```

Cool！！我重载的虚方法被调用了两次，告诉我有2次字段的访问。

在.NET 3.5中提供了Lambda的结构描述类，但我是没有找到动态编译分析的类。看博客园的朋友说：本来有个DynamicExpression的类提供了动态编译功能，但是现在的.NET 3.5砍掉了，但是在Linq 101 samples中却有源代码，靠。

**注意：本程序使用IronPython 的1.0版本，2.0版本的方法已经不同。** 下面是2.0的代码：

```csharp
class Program {

        static void Main(string[] args) {

            //引擎

            IronPython.Hosting.PythonEngine engine = IronPython.Hosting.PythonEngine.CurrentEngine;

            //代码单元

            SourceCodeUnit unit = new SourceCodeUnit(engine, "ActiveObject.Contact.Address + 'ds'");

            //上下文和选项

            CompilerContext context = new CompilerContext(unit);

            IronPython.PythonEngineOptions option = new IronPython.PythonEngineOptions();

            //分析表达式

            Parser p = Parser.CreateParser(context,option);

            IronPython.Compiler.Ast.Expression ex = p.ParseExpression();

            //递归查找

            MyWalker w = new MyWalker();

            ex.Walk(w);

        }

    }

    class MyWalker : PythonWalker {

        public override bool Walk(MemberExpression node) {

            Console.WriteLine("Walk:{0},{1}", node.Name, node.Target);

            return base.Walk(node);

        }

    }
```
