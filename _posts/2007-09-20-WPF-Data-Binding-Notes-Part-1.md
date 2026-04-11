# WPF 学习笔记（数据绑定篇）

> 原文：https://www.cnblogs.com/tansm/archive/2007/09/20/900589.html

此文章是一篇学习笔记，是学习Windows SDK中关于数据绑定部分的笔记。如果你安装了Windows SDK，你一般可以在他的例子目录中找到一个叫 ConnectedData 的目录。

# ADODataSet

此示例演示了：

l  如何将DataSet中的一个表绑定了ListBox中；

l  指定明细的显示方式；

l  自定义转换。

![](/images/2007-09-20-WPF-Data-Binding-Notes-Part-1-1.jpg)

要运行此示例，需要运行CopyData.cmd，此批处理复制BookData.mdb到ApplicationData目录。

首先初始化一个DataSet数据源，然后调用：

myListBox.DataContext = myDataSet;

表示为myListBox设置数据上下文为此DataSet，在XML中，描述为：

其中，ItemSource=”{Binding Path=BookTable}”，表示绑定到子属性（在这里是DataSet的子表）BookTable。

ItemTemplate  ="{StaticResource BookItemTemplate}指定了明细的显示模板为静态资源BookItemTemplate，此静态资源的定义如下：

描述了明细使用3列显示，第一列绑定到Title属性，第二列绑定到ISBN属性，第三列绑定到NumPages属性。

第三列的Background也绑定到NumPages属性中，但是你知道的，背景色怎么可以是数字类型呢？这里是指定了一个转换器，转化器是一个实现IValueConverter接口的对象。参考：IntColorConverter.cs。

# BindConversionMarkup

此示例演示了：

l  转换器在转换时获取被转化的类型。

![](/images/2007-09-20-WPF-Data-Binding-Notes-Part-1-2.jpg)

在红色的标签描述中：

前景色和文本都绑定到TheDate属性上，而且他们使用了相同的转换器，那么转换器如何在绑定时，正确的将文本返回给Text属性，而将颜色返回给Foreground属性呢？

```
public class MyConverter : IValueConverter

  {

    public object Convert(object o, Type type,

        object parameter, CultureInfo culture)

    {

        DateTime date = (DateTime)o;

        switch (type.Name)

        {

            case "String":

                return date.ToString("F", culture);

            case "Brush":

                return Brushes.Red;

            default:

                return o;

      }

    }

      public object ConvertBack(object o, Type type,

          object parameter, CultureInfo culture)

      {

          return null;

      }

  }
```

可以看出，Convert通过检测被转换类型，将返回正确的数据，也就是说，绑定系统会正确的转递他希望的数据类型。

# BindDPtoDP

此示例演示了如何绑定到一个已有的控件上。

![](/images/2007-09-20-WPF-Data-Binding-Notes-Part-1-3.jpg)

在此示例中，当我们下拉选择不同的颜色时，下面的正方形颜色会随之改变。下面是绑定代码：

在此代码中，可以设置绑定对象是一个当前的对象，使用ElementName指定，Path属性还是一样的。

当然，此示例也展示了绑定系统的自动转化功能，其数据源是字符串类型，目标被转化为一个颜色刷。
