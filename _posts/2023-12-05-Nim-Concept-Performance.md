## Nim 概念 Concept 对性能的影响
继上一篇文章《[C# 泛型编译特性对性能的影响](https://tansm.github.io/2023/11/29/CSharp-Generic-Performance.html)》后，我又研究了 Nim 语言相关的设计，由于 Nim 语言与 C# 语言有些差异，比如Nim 没有接口，也没有直接的 class 关键字，所以某些实现是变通的办法。

### 概念 Concept
在Nim中没有 Interface 的概念，虽然有多次提案，但似乎语言设计者一直拒绝这一设计，他们提供了 Concept 这一方案。他与C#的interface的相似点包括：
- `concept`和`interface`都用于定义抽象类型和行为规范，允许在没有提供具体实现的情况下定义方法签名。
- 两者都支持多态，允许不同的类型实现相同的接口或概念，并以相同的方式被操作或使用。

但也有不同之处，比如：
- 在Nim的concept中，方法声明不需要关键字（如fn），而是直接指定方法的签名。在C#中，interface中的方法声明需要使用method或property等关键字。
- concept在Nim中比interface更加灵活。concept可以用于限制多个类型的行为，但不一定需要显式声明实现。它允许在函数中使用未显式声明为实现concept的类型。interface在C#中更加严格，要求类型明确地声明并实现接口中的所有成员。
- 在C#中，interface可以在字段、方法参数、方法返回值等多个地方使用，允许动态地访问实现。在Nim中，concept主要用于参数约束，只允许在函数或过程中使用，`无法在字段或返回值中声明`。

下面是这个案例中声明 Concept 的例子：
```nim
type
    IValueGetter = concept s
        s.getValue(int32) is int64
```
有了 concept，我们就可以在泛型中约束 泛型参数的行为，例如下面的泛型类型 MyTestClass[T] ，其类型声明中没有约束 T，但在相关的方法中，允许约束。
```nim
type
    MyTestClass[T] = object
        valueGetter: T

proc run[T: IValueGetter](this: MyTestClass[T]): int64 =
    var r = 0i64
    let n = high(int32) - rand(100).int32
    for i in 0 ..< n:
      r += this.valueGetter.getValue(i)
    return r
```

### Nim 中 Struct 和 Class
在Nim语言中， struct 是使用 object 关键字，同样的，如果在object 的前面加上 ref 关键字，就变成了 struct 引用，自然就相当于 class 了,下面是示例代码：
```nim
type
    StructValueGetter = object
    StructValueGetter2 = object
        someField: int

    ValueGetter = ref object of RootObj
    ClassValueGetter1 = ref object of ValueGetter
    ClassValueGetter2 = ref object of ValueGetter
```
在这个例子中，我们看见如何声明 Struct，以及声明字段， 同时也使用 ref object 声明了 class ，他使用 of 关键字表示继承关系。

你也许注意到，RootObj ，他是nim语言提供的默认基类，你可以理解为c#和java 的object 基类，但nim语言具有很强的灵活性，可以不继承任何基类，或使用其他的基类。

### Nim中定义方法或动态方法
一般情况下，Nim建议使用 proc 和 func 作为方法前的关键字，因为这样的话方法就是静态编译的，具有最佳的性能，func 其实是 proc 的特例，即表示无副作用的 proc，比如你仅仅是读取对象的数据，不会破坏结构，这就是无副作用。
```nim
# 实现 IValueGetter 的 concept
func getValue(this: StructValueGetter, index: int32): int64 =
    result = index.int64 + 3i64

func getValue(this: StructValueGetter2, index: int32): int64 =
    result = index.int64 + 5i64

func getValue(this: ClassValueGetter1, index: int32): int64 =
    result = index.int64 + 5i64

func getValue(this: ClassValueGetter2, index: int32): int64 =
    result = index.int64 + 7i64
```
为实现诸如 c# 的虚方法功能（即动态调用），nim 设计了 method 关键字。
```nim
method getValueCore(this: ValueGetter, index: int32): int64 {.base.} =
    quit "to overrdie"

method getValueCore(this: ClassValueGetter1, index: int32): int64 =
    return getValue(this,index)

method getValueCore(this: ClassValueGetter2, index: int32): int64 =
    return getValue(this,index)

# 为 ValueGetter 实现契约，相当于使用契约调用的是虚方法。
proc getValue(this: ValueGetter, index: int32): int64 =
    return getValueCore(this, index)
```

### 测试
下面我们将使用 stuct 和 class 两种形式，确定 concept 和 泛型结合，对性能的影响。同时我们也观察动态方法调用，即虚方法的调用对性能的影响。
```nim
import std/[times],std/random
type
    IValueGetter = concept s
        s.getValue(int32) is int64

    MyTestClass[T] = object
        valueGetter: T

proc run[T: IValueGetter](this: MyTestClass[T]): int64 =
    var r = 0i64
    let n = high(int32) - rand(100).int32
    for i in 0 ..< n:
      r += this.valueGetter.getValue(i)
    return r

type
    StructValueGetter = object
    StructValueGetter2 = object
        someField: int

    ValueGetter = ref object of RootObj
    ClassValueGetter1 = ref object of ValueGetter
    ClassValueGetter2 = ref object of ValueGetter

# 实现 IValueGetter 的 concept
func getValue(this: StructValueGetter, index: int32): int64 =
    result = index.int64 + 3i64

func getValue(this: StructValueGetter2, index: int32): int64 =
    result = index.int64 + 5i64

func getValue(this: ClassValueGetter1, index: int32): int64 =
    result = index.int64 + 5i64

func getValue(this: ClassValueGetter2, index: int32): int64 =
    result = index.int64 + 7i64

# 为接口 ValueGetter 实现契约，相当于使用契约调用的是委托。
method getValueCore(this: ValueGetter, index: int32): int64 {.base.} =
    quit "to overrdie"

method getValueCore(this: ClassValueGetter1, index: int32): int64 =
    return getValue(this,index)

method getValueCore(this: ClassValueGetter2, index: int32): int64 =
    return getValue(this,index)

proc getValue(this: ValueGetter, index: int32): int64 =
    return getValueCore(this, index)

proc measureTime(caption: string, procToMeasure: proc(): int64) =
  var startTime = cpuTime()
  let r = procToMeasure()
  var endTime = cpuTime()
  echo caption, " time = ", endTime - startTime, " result = ", r

# 运行测试
proc main() =
    randomize()
    let t1 = MyTestClass[StructValueGetter](valueGetter : StructValueGetter())
    measureTime("StructValueGetter ", proc ():int64 = t1.run() )
    let t2 = MyTestClass[ClassValueGetter1](valueGetter : new(ClassValueGetter1))
    measureTime("ClassValueGetter1 ", proc ():int64 = t2.run() )
    let t3 = MyTestClass[ClassValueGetter2](valueGetter : new(ClassValueGetter2))
    measureTime("ClassValueGetter2 ", proc ():int64 = t3.run())
    let t4 = MyTestClass[ValueGetter](valueGetter : new(ClassValueGetter1))
    measureTime("IValueGetter-1    ", proc ():int64 = t4.run())

    let t5 = MyTestClass[ClassValueGetter1](valueGetter : new(ClassValueGetter1))
    measureTime("ClassValueGetter1 ", proc ():int64 = t5.run())
    let t6 = MyTestClass[StructValueGetter2](valueGetter : StructValueGetter2())
    measureTime("StructValueGetter2", proc ():int64 = t6.run())


when isMainModule:
  main()
```

编译的命令行如下：
```
nim c -d:release -x --checks:off  -r ".\demo1.nim"
```
请自己编译时将 demo1.nim 替换成自己的文件名，同时你也可能注意到 我关闭了检查，我发现如果不关闭的话，性能会慢很多。在我的笔记本电脑的参考结果如下：

```text
StructValueGetter  time = 0.429 result = 2305842997402533900
ClassValueGetter1  time = 0.421 result = 2305842825603845693
ClassValueGetter2  time = 0.421 result = 2305842971632730244
IValueGetter-1     time = 4.74 result = 2305842913650672596
ClassValueGetter1  time = 0.4210000000000003 result = 2305842870701000726
StructValueGetter2 time = 0.4190000000000005 result = 2305842920093123411
```

我们注意到：
- 在nim语言实现中 , struct 和 class 在泛型和 concept 的环境下，性能几乎没有影响；
- 与c#不同，nim仍然为每种 class 编译不同的程序，所以性能不会变化；
- 尽量避免使用虚方法，性能的损失很大；