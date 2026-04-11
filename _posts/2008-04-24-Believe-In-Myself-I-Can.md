---
title: "相信自己，我能"
date: 2008-04-24 00:00:00 +0800
---

# 相信自己，我能

今天，是个值得纪念的日子，因为我编写的ORM（名为Torridity）工具经过性能测试，其读取性能全面压倒DataSet、LinQ和DataEntity Framework。

![](/images/2008-04-24-Believe-In-Myself-I-Can-1.jpg)

![](/images/2008-04-24-Believe-In-Myself-I-Can-2.jpg)从截图中你可以看出，我们比DataSet快2倍，比LinQ快4倍，比Entity Framework beta 3快14倍。

测试采用读取1万笔记录，而且为公正起见，所有的计时开始前，都预先读取一笔其他的记录，保证不是连接缓冲池造成的误差。而且测试代码保证都是各自工具最佳的读取方式。

你可以下载附件中的测试程序，但非常的遗憾，Torridity属于公司的财产，所以我不能包含任何关于此部分的代码。要测试程序请：

  - 使用Visual Studio 2008打开项目；

  - 下载Entity Framework beata 3；

我将在随后的时间测试保存、插入和删除功能的性能。

[LinQVsTorridity.rar](https://files.cnblogs.com/tansm/LinQVsTorridity.rar)

请不要发表诸如以下的意见：都没有实际程序在那吹吧。

我想说的是，这个文章仅是为了纪念我的成果，证明自己的能力，以此纪念。
