# WPF学习笔记(数据绑定篇2)

> 原文：https://www.cnblogs.com/tansm/archive/2007/09/21/901122.html

接上篇文章[WPF 学习笔记（数据绑定篇）](http://www.cnblogs.com/tansm/archive/2007/09/20/900589.html)

# BindToMethod
 这个示例功能是将标准温度转化为：摄氏度Celsius或者华氏度Fahrenheit，此转化是通过调用一个public string ConvertTemp(double degree, TempType temptype)方法获得最终的结果的。

 ![](/images/2007-09-21-WPF-Data-Binding-Notes-Part-2-1.png)

 此示例相对还是比较复杂的，首先声明一个方法类型的数据源：

0
Celsius

此XAML描述了使用TemperatureScale类实例的ConvertTemp方法作为数据源，当然，我们需要给一个默认的调用数据。

然后我们希望将文本框的内容绑定到方法的第一个参数上：

这里需要指定将字符串转化为double的转化器，而且还要有一个防止用户录入错误的数据的校验器。

第二步，我们需要将下拉框绑定到第二个参数：

Celsius
Fahrenheit

和上面的一样，绑定到第二个参数上。

这里有个巧妙的地方，XAML直接将对象实例Celsius和Fahrenheit作为ComboBox的明细了，我们看见下拉框显示正确，而且可以不用转换器直接作为第二个参数。

最后就是绑定结果了:

# HierarchicalDataTemplate
 在这里例子中，我们将看见：

 - 具有层次结构的数据绑定到树控件和菜单控件；
- 在延伸的代码中，你将看见这种绑定同样支持事件跟踪。

 [![](/images/2007-09-21-WPF-Data-Binding-Notes-Part-2-2.png)](http://www.cnblogs.com/images/cnblogs_com/tansm/WindowsLiveWriter/WPF2_8F23/image%7B0%7D%5B15%5D.png)

 在此示例中，包含一个层次结构的数据对象：

 [![](/images/2007-09-21-WPF-Data-Binding-Notes-Part-2-3.png)](http://www.cnblogs.com/images/cnblogs_com/tansm/WindowsLiveWriter/WPF2_8F23/image%7B0%7D%5B18%5D.png)

 在绑定到树控件和菜单时，是直接绑定的：

 那么控件如何知道他的明细使用哪个集合属性呢？这里就靠HierarchicalDataTemplate了。

上面的XAML作为资源定义，描述为类型League的明细使用Divisions属性。

 最后我对这个示例程序进行了改造，首先修改数据类的所有List类型改为ObservableCollection，以便集合实现更改通知。然后我添加了一个菜单：AddTeam，单击此菜单时，我向数据源添加了一个Team对象，我发现绑定系统都自动检测到这个改变了。

 因此，我们可以利用此特性，定义一个描述菜单的实体结构，然后绑定到菜单上，这样，实际的代码将操控与特定菜单实现无关的实体，从而达到隔离的目的。
