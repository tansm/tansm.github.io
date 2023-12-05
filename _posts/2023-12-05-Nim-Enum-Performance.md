## Nim 枚举类型 对性能的影响
继上一篇文章《[Nim 概念 Concept 对性能的影响](https://tansm.github.io/2023/12/05/Nim-Concept-Performance.html)》后，我在想，既然 method 虚方法造成性能的影响很大，那么有没有更快的方法实现，当然是有的，那就是枚举类型。

### Enum Type
与很多的新设计的一样，Nim语言也内置了枚举类型，比如下面的代码：
```nim
type
    ValueGetterKind = enum
        vkClass1,
        vkClass2
    ValueGetter2 = ref object
        sharedField : int32
        case kind:ValueGetterKind
        of vkClass1:someField1 : int32
        of vkClass2:someField2 : int32
```

正如你看到的，在Nim语言中，通过为类型增加一个枚举字段（这里是kind），就可以为不同的类型定义不同的字段了，当然，你也可以定义共享的字段（这里例子的 sharedField）。

官方的例子中，演示了更复杂的情况，比如 多个类型公用相同的字段，或者定义多个字段。

```nim
# This is an example how an abstract syntax tree could be modelled in Nim
type
  NodeKind = enum  # the different node types
    nkInt,          # a leaf with an integer value
    nkFloat,        # a leaf with a float value
    nkString,       # a leaf with a string value
    nkAdd,          # an addition
    nkSub,          # a subtraction
    nkIf            # an if statement
  Node = ref object
    case kind: NodeKind  # the `kind` field is the discriminator
    of nkInt: intVal: int
    of nkFloat: floatVal: float
    of nkString: strVal: string
    of nkAdd, nkSub:
      leftOp, rightOp: Node
    of nkIf:
      condition, thenPart, elsePart: Node

var n = Node(kind: nkFloat, floatVal: 1.0)
# the following statement raises an `FieldDefect` exception, because
# n.kind's value does not fit:
n.strVal = ""
```

### 性能对比
接着我们之前的例子，我们看看在泛型的情况下，调用枚举类型对性能究竟影响有多少。

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
    ValueGetterKind = enum
        vkClass1,
        vkClass2,
        #vkClass3,
    ValueGetter2 = ref object
        sharedField : int32
        case kind:ValueGetterKind
        of vkClass1:someField1 : int32
        of vkClass2:someField2 : int32
        #of vkClass3:someField3 : int32

func getValue(this: ValueGetter2, index: int32): int64 =
    #return index.int64 + 7i64
    case this.kind:
    of vkClass1: 
        result = index.int64 + 9i64
    of vkClass2:
        result = index.int64 + 11i64
    #of vkClass3:
    #    result = index.int64 + 17i64

proc measureTime(caption: string, procToMeasure: proc(): int64) =
  var startTime = cpuTime()
  let r = procToMeasure()
  var endTime = cpuTime()
  echo caption, " time = ", endTime - startTime, " result = ", r

# 运行测试
proc main() =
    randomize()

    let t7 = MyTestClass[ValueGetter2](valueGetter : ValueGetter2(kind:vkClass1, someField1:9))
    measureTime("ValueGetter2:1    ", proc ():int64 = t7.run())
    let t8 = MyTestClass[ValueGetter2](valueGetter : ValueGetter2(kind:vkClass2, someField2:11))
    measureTime("ValueGetter2:2    ", proc ():int64 = t8.run())
    #let t9 = MyTestClass[ValueGetter2](valueGetter : ValueGetter2(kind:vkClass3, someField3:11))
    #measureTime("ValueGetter2:3    ", proc ():int64 = t9.run())   

when isMainModule:
  main()

```

在我的机器中，测试结果如下（均使用release编译，下同）：
```text
ValueGetter2:1     time = 0.725 result = 2305842980222664759
ValueGetter2:2     time = 1.967 result = 2305842982370148375
```
我们看到，如果是判断分支的第一个，性能与本地调用稍差（0.7对比0.4），但与虚方法的5秒已经好太多。

细心的你，可能已经注意到我给的代码注释掉了第三种类型vkClass3，那么是不是作为分支的第三种情况，会更慢？让我们取消几个分支的注释，看看结果：
```text
ValueGetter2:1     time = 1.300 result = 2305842894323320179
ValueGetter2:2     time = 1.345 result = 2305842842783714180
ValueGetter2:3     time = 1.295 result = 2305842993107566484
```
意不意外？第一个分支竟然变慢了，而后两个分支变快了，最后一个分支还稍稍快一点点。是否说明nim语言编译为2个枚举类型和3个枚举类型，使用了不同的编译代码？ 让我们从nim生成的c语言源码验证这件事。
```c
// 这是2个枚举的代码
N_LIB_PRIVATE N_NIMCALL(NI64, getValue__demo50_u28)(tyObject_ValueGetter2colonObjectType___liWfIs6eMPnbT0BuR4LSgg* this_p0, NI32 index_p1) {
	NI64 result;
	result = (NI64)0;
	switch ((*this_p0).kind) {
	case ((tyEnum_ValueGetterKind__tsIJjFnfs8zo9caQCLYJn1A)0):
	{
		result = (NI64)(((NI64) (index_p1)) + IL64(9));
	}
	break;
	case ((tyEnum_ValueGetterKind__tsIJjFnfs8zo9caQCLYJn1A)1):
	{
		result = (NI64)(((NI64) (index_p1)) + IL64(11));
	}
	break;
	}
	return result;
}
```

```c
// 这是三个枚举的代码
N_LIB_PRIVATE N_NIMCALL(NI64, getValue__demo50_u30)(tyObject_ValueGetter2colonObjectType___liWfIs6eMPnbT0BuR4LSgg* this_p0, NI32 index_p1) {
	NI64 result;
	result = (NI64)0;
	switch ((*this_p0).kind) {
	case ((tyEnum_ValueGetterKind__tsIJjFnfs8zo9caQCLYJn1A)0):
	{
		result = (NI64)(((NI64) (index_p1)) + IL64(9));
	}
	break;
	case ((tyEnum_ValueGetterKind__tsIJjFnfs8zo9caQCLYJn1A)1):
	{
		result = (NI64)(((NI64) (index_p1)) + IL64(11));
	}
	break;
	case ((tyEnum_ValueGetterKind__tsIJjFnfs8zo9caQCLYJn1A)2):
	{
		result = (NI64)(((NI64) (index_p1)) + IL64(17));
	}
	break;
	}
	return result;
}
```
可以看出，代码完全是一样的，可能是编译器做了手脚。当然，我也测试了更多枚举的可能，比如7种，我发现所有分支时间都变长了（达到2秒），但总体是比虚方法好的。

### 共享的字段
在 getValue 方法中，如果你没有对类型进行判断，那么实际测试的成绩和静态方法调用是一样的，这可以被利用到我们面向对象编程中，经常用到的基类统一实现的场景。

### 总结
在 nim 编程中，简单的派生关系可以使用 枚举类型来提高性能。