---
layout: post
title: "WPF绑定技术一步步学"
date: 2006-11-24 08:00:00 +0800
---

> 原文：https://www.cnblogs.com/tansm/archive/2006/11/24/571097.html
作者：谈少民

编写日期：2006-11-24

摘要：本文通过一个实例帮助读者了解WPF基本的绑定操作，包括绑定到实体、集合、格式化、校验等任务。

要求：阅读本文需要对WPF有个基本的认识，熟悉.NET开发。

# 开始

首先下载本文章的例子（如果你安装了Vista SDK应该已经包含了这个例子），他是MS为WPF创建的一个学习例子，地址是：

[http://download.microsoft.com/download/9/d/0/9d0d2396-3219-4d4c-a7dd-b26c8ad400b7/WPFSamples.exe](http://download.microsoft.com/download/9/d/0/9d0d2396-3219-4d4c-a7dd-b26c8ad400b7/WPFSamples.exe)

安装后，你可以在以下目录下找到这个例子：

C:\Program Files\Microsoft SDKs\Microsoft SDKs\Windows\v6.0\Samples\ConnectedData\DataBindingLab

OK，首先我们直接运行这个例子，运行的样子如下：

 ![](/images/2006-11-24-Step-by-Step-Learning-WPF-Binding-1.PNG)

如果点击 Add Product按钮，将出现下面的画面：

 ![](/images/2006-11-24-Step-by-Step-Learning-WPF-Binding-2.PNG)

# 数据源

在.NET 2.0中，可以直接使用和常见的数据源包括：DataSet和BindingSource，事实上,还是比较宽松的,单个对象的属性都可以绑定，如果要做到同步，包含 ***PropertyName***Changed事件或者符合System.ComponentModel.INotifyPropertyChanged接口就可以了，在集合绑定方面只要符合System.Collections.ICollection接口就可以，问题在如果要实现跟踪，就只有System.ComponentModel.IBindingList可以选择了，这个接口设计的有些大了。

.NET 3.0中，对集合更新的跟踪做了简单的定义：System.Collections.Specialized.INotifyCollectionChanged，而且还有一个System.Windows.IWeakEventListener接口，看名字就知道了。

现成的绑定源我找到有System.Windows.Data.CollectionViewSource和System.Collections.ObjectModel.ObservableCollection，好像选择挺多的，有点晕了。

CollectionViewSource是个很Cool的组件，他可以让你分组或者排序，非常的方便。

# 绑定

先拿添加产品那个窗口为例子，因为他足够简单，只需要绑定到普通对象上，下面是绑定到Description的文本框：

```text

```
意思是Text属性使用绑定功能，使用Binding对象处理，其中绑定到对象上的属性为Description，更新模式是PropertyChanged模式。

上面的描述也可以用另外一种语法：

```text

```
道理一样的。

因为绑定指定的是Path而不是PropertyName，所以你也可以写成Owner.Name：

```text

```
如果你足够有想象力的话，集合也可以绑定，象这样：

```text
Path=(Validation.Errors)[0].ErrorContent
```
现在指定了绑定到哪个属性，那如何指定数据源呢？控件上（实际上是FrameworkElement）包含一个重要的属性DataContext，意思是数据上下文（废话），你的绑定数据都来源这里，我感觉他象是内置的BindingSource控件。

在初始化时，AddProduct窗口包含下面的代码：

```text
private void OnInit(object sender, RoutedEventArgs e)

{

this.DataContext = new AuctionItem("Type your description here",

ProductCategory.DVDs, 1, DateTime.Now, ((DataBindingLabApp)Application.Current).CurrentUser,

SpecialFeatures.None);

}
```

这就等于更新了数据源的内容。

你也可以使用命名的数据源，他作为一种资源出现：

```text

```
他使用了CollectionViewSource作为数据源控件，内部跟踪Application.Current.AuctionItems的数据，在这个例子中AuctionItems是一个ObservableCollection集合类实例。

在使用时，指定他就是了：

```text

```

## 数据转化

在这个例子中，包含了一个日期和字符串互相转化的例子，第一步工作是编写一个转化器类：

```csharp
[ValueConversion(typeof(DateTime), typeof(String))]

public class DateConverter : IValueConverter

{

public object Convert(object value, Type targetType, object parameter, CultureInfo culture)

{

DateTime date = (DateTime)value;

return date.ToShortDateString();

}

public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)

{

string strValue = value.ToString();

DateTime resultDateTime;

if (DateTime.TryParse(strValue, out resultDateTime))

{

return resultDateTime;

}

return value;

}

}
```
这个转化类必须符合IValueConverter接口，并且在类上部标记ValueConversion标记，指明是DateTime和String之间的转化，这个功能是.NET 3.0新增的功能，位于System.Data.Windows命名空间。

要应用这个转化工具还真的挺绕的，首先需要将其定义成一个命名资源，这个申明放在Application中：

```text

```
然后，你就可以在绑定的文本框中使用他了，例如在添加产品的Start Date中：

```text

```
注意绑定申明中的Converter属性的指定。

另外，还有一种IMultiValueConverter，

# 数据校验

数据校验是另外一个重要的功能，在.NET 2.0中，数据源只需要符合System.ComponentModel.IDataErrorInfo接口就可以了，DataSet就是一个例子，在.NET 3.0中使用了ValidationRule抽象类处理校验工作，他将被依附在Binding中。

```csharp
class FutureDateRule : ValidationRule {

public override ValidationResult Validate(object value, CultureInfo cultureInfo) {

DateTime date;

try {

date = DateTime.Parse(value.ToString());

}

catch (FormatException) {

return new ValidationResult(false, "Value is not a valid date.");

}

if (DateTime.Now.Date > date) {

return new ValidationResult(false, "Please enter a date in the future.");

}

else {

return ValidationResult.ValidResult;

}

}

}
```
还是比较简单的，没有什么可解释的，一样，这个校验器需要具体使用到哪个Binding中，

```text

```
你可能注意到你是可以指定多个校验器。

参考资料：

[Windows Presentation Foundation 数据绑定：第一部分](http://www.microsoft.com/china/MSDN/library/Windev/WindowsVista/WPFDataBinding_Pt1.mspx?mfr=true)

[Windows Presentation Foundation 数据绑定：第二部分](http://www.microsoft.com/china/MSDN/library/Windev/WindowsVista/WPFDataBinding_Pt2.mspx)
