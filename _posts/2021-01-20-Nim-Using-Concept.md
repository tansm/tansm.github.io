# nim 语言使用 concept 实现 c# 的interface

> 原文：https://www.cnblogs.com/tansm/p/nimUsingConcept.html

## nim简介

nim语言兼顾C#等高级语言语义表达的丰富性，又有 C 语言的灵活性，以及[超强的性能](https://github.com/frol/completely-unscientific-benchmarks)。下面是中文站对他的总结，我抄下来：

## Nim 是一种静态类型的、编译型、系统编程语言。
它结合了其他成熟语言的成功概念。
（如 Python、Ada 和 Modula）

### 效率

- Nim 生成原生且无依赖的可执行文件，不依赖于虚拟机，
所以它们小巧易分发。

- Nim 编译器和生成的可执行文件，对目前的任何主流平台都提供了支持，
包括 Windows、Linux、BSD 和 macOS。

- Nim 的内存管理是确定性的，可使用析构函数和移动语义进行自定义， 其灵感来自C++和Rust。 非常适合嵌入式硬实时系统。

- 零开销迭代器和用户自定义方法的编译期求值等现代概念，
结合优先使用分配在栈上的值类型数据，生成高性能代码。

- 支持各种后端：可以被编译为 C、C++ 或 JavaScript， 以便 Nim 可用于所有后端和前端需求。

### 表现力

- Nim 实现了自举：编译器和标准库都是用 Nim 本身来实现的。

- Nim 拥有强大的宏系统，允许直接操纵 AST，提供无限的可能性。

### 优雅

- 宏不会改变 Nim 的语法，因为并没有这个必要
—— Nim 语法本身已经足够灵活。

- 具有局部类型推断、元组、泛型和sum类型的现代类型系统。

- 语句按缩进分组，也可以跨行。

## 接口

我满怀信心的开始学习 nim 语言，之前有 c# 和 kotlin 的编程经验，所以自然会将面向对象的思路带到 nim 中，看官方文档，[继承功能完全是内建的](https://nim-lang-cn.org/docs/tut2.html#%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E7%BC%96%E7%A8%8B)，所以没有遇到什么困难，结果在 接口 interface 问题上卡壳了。

查询很多的资料，有几种解法，一种是官方库中 [stream](https://nim-lang.org/docs/streams.html#Stream) 的解法，要定义一个 stream 和一个 streamObj ，然后在实现类中一一映射，我觉得太麻烦，直接放弃了。

还有一篇[文章介绍了利用 nim 的宏功能](https://forum.nim-lang.org/t/2741)，自己造了[一个轮子](https://github.com/zielmicha/collections.nim/blob/master/collections/iface.nim)，我觉得主要的问题是，不是官方支持的，我如果写了大量的代码，有问题不好解决，而且有更好的官方方案的话，就要大量重写。况且，他构建了 vTable ,有间接调用成本。

但是，功夫不负有心人，我终于找到官方的解决方案，使用 concept 来解决 interface 的问题， concept 中文翻译为 概念，有点拗口，我的理解是，定义一堆约束，如果某个类符合这个约束，那么就自然可以使用这个约束来访问，这不就是 interface 想要表达的意思吗？

但我觉得 nim 设计 concept 时，思想比 interface 更进一步，就是[鸭子类型（duck typing）](https://www.cnblogs.com/moqiutao/p/13899358.html)的概念，即如果它走起路来像鸭子，叫起来也是鸭子，那么它就是鸭子。

下面是定义 concept （接口） 的方法：

```text
type
# 定义一个 IImmutableList<E> 这样的接口，在nim中叫 概念
    IImmutableList[E] = concept self

        # 包含一个方法，允许用 var a = obj[32] 这样的方式索引访问。
        self.`[]`(uint64) is E

        # 定义了一个 size 只读属性
        self.size is uint64
```

然后你就可以写自己的类了，只要你的方法和签名与接口一致就可以了，不需要像 c# 或 kotlin ,java 那样要显式的写实现了此接口。

```text
type
    SeqImmutableList[E] = ref object of RootObj
        items : seq[E]

# 为 SeqImmutableList 定义了一个 size 属性
proc size*[E](this: SeqImmutableList[E]) : uint64 {.inline.} =
    uint64(this.items.len) #用到类型转换

# 定义一个可以直接使用 [] 方式访问的方法。
proc `[]`*[E](this: SeqImmutableList[E], index : uint64) : E {.inline.} =
    this.items[int(index)]

# 相当于构造函数，只有构造函数才能访问私有变量。
proc newSeqImmutableList*[E](items : seq[E]) : SeqImmutableList[E] =
    new(result)
    result.items = items

let target = newSeqImmutableList(@[1,2,3,4])
echo "size = ", target.size
echo "target[2] = ", target[2]
```

到目前，上面的代码仍然是类自己的方法去访问的，但是你想使用接口的方式访问，非常的方便，直接传递就可以了。

```text
# 以接口（概念）的方式访问
proc printAll(this: IImmutableList) =
    echo "size = ", this.size
    for i in 0..<this.size:
        echo "items[", i, "] = ", this[i]

printAll(target)
```

这里我将 target （他是SeqImmutableList<int>类型）直接传递给 printAll 方法，编译器只要觉得符合 概念(concept)，就可以传递。

## 接口的继承

如果我现在想定义一个新的接口，派生自原来的接口怎么办呢？

```text
IMutableList[E] = concept self
        # 相当于c# 的继承
        self is IImmutableList[E]

        # 提供按索引更改的能力，类似 c# 的 this[i] = x 这样的赋值。
        self.`[]=`(idx,E)
```

使用 is ... 表达了继承的概念，然后再编写你新的想定义的方法。

现在，我们新定义一个类，让他两个接口都支持，当然，我们顺便也学习一下对象继承的写法。

```text
type
    SeqMutableList[E] = ref object of SeqImmutableList[E]
        isDirty : bool

# 提供按索引的写
proc `[]=`[E](this: SeqMutableList, index:idx, value : E) =
    this.elements[int(index)] = value

proc newSeqMutableList[E](items : seq[E]) : SeqMutableList[E] =
    return SeqMutableList[E](elements:items, isDirty:false)
```

现在，你可以按照类本身 SeqMutableList[E] 的方式访问他，也可以使用两个接口 IImmutableList[E] 和 IMutableList[E] 的方式，也可以使用基类 SeqImmutableList 的方式访问

```text
# 创建可变的对象
let target2 = newSeqMutableList(@[1,2,3,4,5])
echo "size2 = ", target2.size #访问基类的方法

# 修改他的值，并使用基类提供的方法重新获取
target2[2] = 99
echo "target2[2] = ", target2[2]

# 可以传递给基准的接口去访问。
printAll(target2)

# 可变的接口访问，包含基类的方法。
let target3 : IMutableList[int] = target2
echo "size3 = ", target3.size
target3[2] = 9999
echo "target3[2] = ", target3[2]
echo "target2[2] = ", target2[2]
let target4 : SeqImmutableList[int] = target2
echo "target4[2] = ", target4[2]
```

 

概念除了可以作为接口使用，还可以对泛型约束，更多复杂的使用方法，参见：https://nim-lang.org/docs/manual_experimental.html#concepts
