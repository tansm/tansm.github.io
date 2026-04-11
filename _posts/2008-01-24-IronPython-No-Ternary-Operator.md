---
layout: post
title: "解决IronPython没有三元运算符的问题"
date: 2008-01-24 08:00:00 +0800
---
今天同事使用IronPython中的Lambda写程序（我们的程序使用IronPython的Lambda功能），发现一个问题，假设有函数：c = a / b，可是b有可能为0，如果为0，那么我们希望c=0，由于是Lambda表达式，所以必须使用一行话描述，可惜查资料发现IronPython不支持三元运算符，后来查资料，发现Snowdream兄写了解决方案： [Python 学习笔记 (3)](http://www.blogjava.net/zellux/archive/2007/07/10/129284.html) 。

修改后的程序是：  b!=0 and a/b or 0，注意这里使用了不等于，我们发现b=0时，不会运算a/b，也就起到我们的目的。

2011-8-15 补充

今天再次遇到此类问题，再次搜索网络，发现其实有比较平滑的写法：

Result = A / B if B <> 0 else 0

即如果B不等于0，计算表达式，否则返回0

再次补充：

我好笨啊，已经有人去年就回帖指明了此方法，我还没有注意。呜呜，打屁股。
