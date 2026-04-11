# WPF学习笔记(数据绑定篇3)

> 原文：https://www.cnblogs.com/tansm/archive/2007/09/30/902996.html

接上回的《[WPF学习笔记(数据绑定篇2)](http://www.cnblogs.com/tansm/archive/2007/09/21/901122.html)》，继续

# BindValidation

此示例演示了：

    - 如何使用错误模板；

- 使用样式显示错误信息；

- 如何在校验发生异常时执行回调；

![](/images/2007-09-30-WPF-Data-Binding-Notes-Part-3-1.jpg)

首先，你可以看见XAML中使用自定义的错误模板，指定错误模板的方式是：

此错误模板我简单改造了一下，变得好看点：

注意这个模板是ControlTemplate（为什么是控件模板我也不知道，照葫芦画瓢），然后定义了一个布局，左边一个图像，右边一个AdornedElementPlaceholder占位符。

（我在实验时，图像如果没有加入Width和Height，在显示时图片将变得很大）。

当然，你也可以使用样式绑定到异常上来显示错误，例如：

当然，例子中还显示了，如果校验时发生异常（注意：是异常不是不正确的数据），将发生回调：

```
BindingExpression myBindingExpression = textBox3.GetBindingExpression(TextBox.TextProperty);

          Binding myBinding = myBindingExpression.ParentBinding;

          myBinding.UpdateSourceExceptionFilter = new UpdateSourceExceptionFilterCallback(ReturnExceptionHandler);

          myBindingExpression.UpdateSource();
```

当然，为什么是ParentBinding呢？需要想想看。
