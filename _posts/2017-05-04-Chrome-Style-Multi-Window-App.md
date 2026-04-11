# 将多窗体应用程序改造为仿Chrome形式的简易方法

> 原文：https://www.cnblogs.com/tansm/p/6807060.html

# 需求

在我们现有的ERP应用中，他是基于WinForm设计的，在早期的设计中，我们每打开一个作业，就会新建一个窗口，就像这样：

![Image](/images/2017-05-04-Chrome-Style-Multi-Window-App-1.png)

 当我们打开很多的作业时，用户要通过Windows的任务栏慢慢找到，当然，如果仅仅这个问题，到还能忍受。关键是用户会打开多个客户端，比如一个客户端登录A公司，另外一个客户端登陆B公司，就算我们在标题上添加公司信息，用户也需要时间反应，使用体验并不好。

 

# 解决方案

可能你首先想到MDI窗口，但你知道的，那是年代久远的东西，现在没有多少人愿意看到那个古老的界面风格了。

大多数人希望类似Chrome的风格。

![Image](/images/2017-05-04-Chrome-Style-Multi-Window-App-2.png)

制作这个控件本身不是难点，难点在于如何让以前的旧代码安全的运行？例如旧的代码中总是假设程序是运行在某个WinForm上，比如调用FindForm();

所以我的技巧是：

1、建立一个无边框的窗体（this.FormBorderStyle = System.Windows.Forms.FormBorderStyle.None），放一个TabControl；

2、当需要显示一个子窗口时，将他也设计为无边框，然后嵌入到TabPage中，类似这样：

```text
Form2 f2 = new Form2();
f2.Dock = DockStyle.Fill;
f2.TopLevel = false;
TabPage p2 = new TabPage(f2.Text);
p2.Controls.Add(f2);
tabControl1.TabPages.Add(p2);
f2.Show();
tabControl1.SelectTab(p2);
```

其中f2就是潜入的窗口，这样看起来的效果就类似Chrome了，而且我执行FindForm是没有问题的。

![Image](/images/2017-05-04-Chrome-Style-Multi-Window-App-3.png)

 

[示例代码下载](https://files.cnblogs.com/files/tansm/TabForm.zip)
