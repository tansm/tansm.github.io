# nim 语言实现迭代器

> 原文：https://www.cnblogs.com/tansm/p/nimUsingIterator.html

nim语言默认是支持 for x in items 这样的迭代的，而且一个类如果要支持迭代，可以用 yield 关键字，其实在 nim 主页上第二个例子就已经重点介绍了。

```text
# Thanks to Nim's 'iterator' and 'yield' constructs,
# iterators are as easy to write as ordinary
# functions. They are compiled to inline loops.
iterator oddNumbers[Idx, T](a: array[Idx, T]): T =
  for x in a:
    if x mod 2 == 1:
      yield x

for odd in oddNumbers([3, 6, 9, 12, 15, 18]):
  echo odd
```

但如果我的类希望像数组那样直接被迭代，而不是要调用一个方法，怎么办？ 事实上，nim 语言提供了语法糖，如果你迭代的对象包含一个 items 方法，且返回的是 iterator，那么也可以的

```text
# 让其支持直接的 for 迭代访问，默认情况下，如果 直接 for 一个变量，将默认调用他的items方法。
iterator items[E](this:SeqImmutableList[E]): E =
    for x in this.elements:
        yield x
```

这样你就可以直接迭代了：

```text
let target = newSeqImmutableList(@[1,2,3,4])
for x in target:
    echo x
```

 

如果要使用高级的特性，可以参考语言手册：  https://nim-lang.org/docs/manual.html#iterators-and-the-for-statement

比如，他就提到使用 pairs 这样的方法，实现字典需要迭代出两个参数的需求。

另外，还提到有两种迭代器，内联迭代器 和 闭包迭代器。

 

完整代码请参考：

```text
type
    # 定义一个新的别名，用来统一对大数据结构的索引访问。
    idx = uint64

    SeqImmutableList[E] = ref object
        elements : seq[E]

# 为 SeqImmutableList 定义了一个 size 属性
proc size*[E](this: SeqImmutableList[E]) : idx {.inline.} =
    uint64(this.elements.len) #用到类型转换

# 定义一个可以直接使用 [] 方式访问的方法。
proc `[]`*[E](this: SeqImmutableList[E], index : idx) : E {.inline.} =
    this.elements[int(index)]

# 让其支持直接的 for 迭代访问，默认情况下，如果 直接 for 一个变量，将默认调用他的items方法。
iterator items[E](this:SeqImmutableList[E]): E =
    for x in this.elements:
        yield x

# 相当于构造函数，只有构造函数才能访问私有变量。
proc newSeqImmutableList*[E](items : seq[E]) : SeqImmutableList[E] =
    new(result)
    result.elements = items

let target = newSeqImmutableList(@[1,2,3,4])
echo "size = ", target.size
echo "target[2] = ", target[2]
for x in target:
    echo x
```
