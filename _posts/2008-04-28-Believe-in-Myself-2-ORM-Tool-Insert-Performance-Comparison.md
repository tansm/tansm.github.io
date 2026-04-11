---
title: "相信自己，我能2 — ORM 工具的插入性能比较"
date: 2008-04-28 00:00:00 +0800
---

# 相信自己，我能2 — ORM 工具的插入性能比较

请先参考《[相信自己，我能](http://www.cnblogs.com/tansm/archive/2008/04/24/TorridityVsLinQ.html)》 关于读取性能的测试

今天对插入的性能做了比较，和我预期的一样，Torridity的性能和其他的产品基本打平手，而Entity Framework一样是糟糕的成绩。

为什么认为会打个平手呢？因为插入1万比记录，意味这连续发送1万次SQL，在没有提别糟糕的程序下，基本上SQL语句的执行花去了几乎大部分的时间，程序本身的优化所得到的提升几乎微乎其微。

下面是测试结果：

![](/images/2008-04-28-Believe-in-Myself-2-ORM-Tool-Insert-Performance-Comparison-1.jpg)

再次失望ADO Entity Framwork的性能，太糟糕了
