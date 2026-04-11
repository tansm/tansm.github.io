# WPF 学习笔记（Menu篇）

> 原文：https://www.cnblogs.com/tansm/archive/2007/08/04/842680.html

不学习就要被淘汰！努力。

主菜单使用Menu作为总容器，可以使用MenuItem作为子项目。
MenuItem最重要的属性是Header属性（郁闷，为什么原先统一的Text属性现在搞得又一团糟），在正文中可以使用下划线表示Alt快捷键，例如：文件(_F) ，又是郁闷的事情，为什么不继续使用 文件(&F) 的方式。
要指定快捷键，可以使用：InputGestureText="Ctrl+X" ，另外，菜单的提示也相对简单，仍然是 ToolTip 属性。
要插入一个分割线，是插入一个子项目。
要相应菜单的单击，可以使用 Click="mnuOpen_Click" ，还是很简单。
实验了一下在菜单中插入一个文本框，最终效果是出来了，可是做的还不好，例如光标移动到文本框录入文本时，只要鼠标移走，光标立即就消失，离实用还不够精细（在Orcas beta1下我测试的时候退出程序，VS会崩溃）。
菜单前面的复选框可是使用IsChecked属性控制，点击后自动控制复选框的属性是IsCheckable。菜单的有效性也换了名字，不是Enabled，而是IsEnabled。
