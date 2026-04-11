# C# 性能优化之斤斤计较篇  一

> 原文：https://www.cnblogs.com/tansm/archive/2012/05/13/dotnet_profile1.html

今天，我想跟大家聊一聊C#的性能优化，当然，这里并不谈基本的原则，这些都假设你已经非常精通了，本文聊的是要争取几个毫秒的程序。关于基本的性能优化，可以参考园子里的文章。比如：

[.NET 性能优化方法总结](http://blog.csdn.net/bluedoctor/article/details/3859847)

先说说我的测试环境：

![Image](/images/2012-05-13-CSharp-Performance-Optimization-Part-1-1.png)

一台典型的笔记本电脑，Windows 7中文版，.net Framework用的是4.5版本，VS是现在VS11 beta版。我也是用VS2008这样的环境测试了下面的所有场景，发现没有任何区别，所以就以VS11为基准了。

所有测试数据都是编译为Relase，且不包含PDB，直接双击运行而非在VS环境下执行。[点击这里下载源代码](https://files.cnblogs.com/tansm/DotNetProfile.7z)。

言归正传，先测试第一点：

 

## 静态方法比实例方法快吗？

我们总是从各个渠道听说：静态方法比实例方法要快，所以，我想亲自试试。测试方法很简单，循环调用实例方法和静态方法。

```text
/// <summary>
    /// 这是一个普通类，调用实例的方法
    /// </summary>
    public class C1 {
        public void DoLoop() {
            for (int i = 0; i < int.MaxValue; i++) {
                DoIt();
            }
        }

        private void DoIt() {
        }
    }

    /// <summary>
    /// 使用静态方法调用。
    /// </summary>
    public static class C2 {
        public static void DoLoop() {
            for (int i = 0; i < int.MaxValue; i++) {
                DoIt();
            }
        }

        private static void DoIt() {
        }
    }
```

测试结果如下：

![Image](/images/2012-05-13-CSharp-Performance-Optimization-Part-1-2.png)

测试多次，基本偏差不大，只能说，静态方法比实例方法快那么可怜的一点点，鉴于实例方法的灵活性远大于静态方法，所以还是一般使用实例方法吧。

也实验过，在方法中访问实例字段和静态字段，发现也没有区别，所以不再单独罗列代码。

 

## 避免方法内创建实例的情况

这个要讨论的问题有点难说明，我们还是先看一看.net内部的代码吧,下面是一段Collection<T>.Add的方法：

```text
public void Add(T item)
{
	if (this.items.IsReadOnly)
	{
		ThrowHelper.ThrowNotSupportedException(ExceptionResource.NotSupported_ReadOnlyCollection);
	}
	int count = this.items.Count;
	this.InsertItem(count, item);
}
```

注意ThrowHelper类，如果换成我们自己写，一句话就搞定了：throw new NotSupportedException。为什么微软要这么写呢？

老外有解释：[Why does SortedList implementation use ThrowHelper instead of throwing directly?](http://stackoverflow.com/questions/562479/why-does-sortedlist-implementation-use-throwhelper-instead-of-throwing-directly)

其实，我也是信奉此真理，而且就在前一段时间，一位同事还找我问，两段几乎一样的代码，为什么测试性能有差距，结果我按照此原理，将异常抛出放在外面，结果真的变好了。

现在，我还要再次测试一下，我相信的是数据：

```text
class C1 {
        private Dictionary<int, int> _dict = new Dictionary<int, int>() ;
        public C1() {
            _dict.Add(1, 1);
            _dict.Add(2, 2);
        }

        public void Do1() {
            object obj = new object();
            for (int i = 0; i < int.MaxValue/100; i++) {
                GetItOne(1);
            }
        }

        //这个方法，在内部可能创建实例
        private int GetItOne(int key) {
            int value;
            if (!_dict.TryGetValue(key,out value)) {
                throw new ArgumentOutOfRangeException("key");
            }
            return value;
        }

        public void Do2() {
            for (int i = 0; i < int.MaxValue/100; i++) {
                GetItTwo(1);
            }
        }

        //这个方法，将创建实例的代码移动到外部
        private int GetItTwo(int key) {
            int value;
            if (!_dict.TryGetValue(key, out value)) {
                ThrowArgumentOutOfRangeException();
            }
            return value;
        }

        private static void ThrowArgumentOutOfRangeException() {
            throw new ArgumentOutOfRangeException("key");
        }
    }
```

测试结果是：

![Image](/images/2012-05-13-CSharp-Performance-Optimization-Part-1-3.png)

基本上，会快0.06秒左右，但是如此大的循环得到的好处并不是那么的明显，但有作用。这种写法还是比较舒服的，所以还是建议大家用吧。

 

下篇我将实验：数组的枚举，类和结构创建的成本。
