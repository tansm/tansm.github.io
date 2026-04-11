---
layout: post
title: ".NET 二进制反序列化时遇到的问题"
date: 2008-01-31 08:00:00 +0800
---
这几天对动态实体做了一些修改，然后同事们一运行就出错了，仔细调试后发现是在反序列化时，我写了一个回调，然而发生回调时，对象没有完全初始化完毕。具体是这样的：

我有个类：DependencyMetadataBase，包含一个方法：

```csharp
[OnDeserialized()]

        private void OnDeserialized(StreamingContext context) {

            this.OnDeserialization(null);

        }
```

而且，这个类的派生类包含了一个内部变量：

```csharp
private DependencyPropertyData _data;
```

很不幸，这个反序列化时的回调需要使用_data变量，跟踪发现，当回调发生时，_data不是null，但_data内部的变量中，值类型(包括string）都已经填充，但是引用类型（Class）的都是null，导致了我的回调发生错误。

查看MSDN，得到以下文字：

对象将被彻底重新构造，并且在反序列化期间调用方法可能会产生不良的副作用，因为被调用的方法可能引用在进行此调用时尚未被反序列化的对象引用。如果反序列化的类实现 IDeserializationCallback，则在整个对象图已被反序列化后将自动调用 OnDeserialization 方法。此时，所有引用的子对象均已被完全还原。哈希表就是在不使用事件侦听器的情况下很难反序列化的类的典型示例。在反序列化期间检索键/值对十分容易，但将这些对象添加回哈希表可能会造成一些问题，因为不能保证从哈希表派生的类已被反序列化。因此，不建议在此阶段对哈希表调用方法。

所以，我使用IDeserializationCallback实现了回调功能。事实上，我以前另外一个类就使用了IDeserializationCallback这个方法，但为什么后来这个类又不用呢？当时遇到一个问题，假设A实现了IDeserializationCallback,而B也实现了，很遗憾，当A引用了B并反序列化时，并不能保证B先被调用IDeserializationCallback方法，所以当A的IDeserializationCallback调用了B的数据时，很可能B没有调用IDeserializationCallback,造成程序的错误。

所以在当时，我使用了OnDeserializedAttribute，这个回调要早于Callback方式，但很遗憾，直到现在，我才发现OnDeserializedAttribute更危险。

最终的解决方案是都实现Callback模式，但是在A调用B前，先检查B是否已经调用过Callback，如果没有，提前调用。
