# 何时何地不能使用泛型

> 原文：https://www.cnblogs.com/tansm/archive/2008/01/28/1055649.html

今天看见一篇文章[【C#食谱】【面食】菜单1： 何时何地使用泛型](http://www.cnblogs.com/adaiye/archive/2008/01/27/1055319.html) ，总结的很好，我也总结一下，不过是反过来的：何时何地不能使用泛型。

注：以下未特别注明的话，均表示不能对外展现泛型，内部仍然可以使用泛型。

1、控件上，在控件上public一个泛型的属性，意味着窗体设计器无法打开。同样的，在重用的组件上也意味着不能打开设计器 了；

2、WebService上，据我目前所知，目前的WebService规范是不支持泛型的，不管是对外接口，还是返回/传入的数据类型；

3、COM的互操作，COM是不支持泛型的。

解决方案：

在以上的应用中，我们又如何避免泛型带来的麻烦呢？一般你可以：

1、包装泛型类，使其特例化，不再具有泛型的特征。例如：

public sealed class DependencyPropertyCollection : ReadonlyNamedCollection<DependencyProperty>

2、 定义非泛型的接口，将泛型的返回值或参数使用object方式访问，适合COM操作。

欢迎大家补充。

补充：竟然有人说这是垃圾文章，哎，算了，不要丢人了，不要放在首页了。
