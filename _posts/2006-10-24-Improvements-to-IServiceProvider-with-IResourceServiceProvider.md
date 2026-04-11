---
title: "谈IResourceServiceProvider对IServiceProvider的改进"
date: 2006-10-24 15:08
---
IServiceProvider

是一种常见的对服务定位的描述。

![](/images/2006-10-24-Improvements-to-IServiceProvider-with-IResourceServiceProvider-1.gif)

他认为，在一个容器中，对于某种服务来说是唯一的，例如不可能存在两个剪贴板服务，而且这也屏蔽了对服务位置的关心。这种设计对于工具类软件十分有效，例如各种
设计器
软件，但对于具有庞大且重复性很强的数据库类软件来说，他是不太适合的。

例如，订单模块中包含了

Create

、

Read

、

Save

和

Delete

方法，而客户模块同样包含了

CRUD

操作，最简单的办法是建立

IDocumentService

接口，包含了

CRUD

操作，但由于

IServiceProvider

无法为你区分你是申请订单的还是客户模块的，所以你必须从

IDocumentService

中派生出两个新的接口。

![](/images/2006-10-24-Improvements-to-IServiceProvider-with-IResourceServiceProvider-2.gif)

但是你要知道，现在的数据库软件动辄

1000

多个模块，这个工作量非常巨大，而且他还有致命的弱点：你无法在运行时通过模块的名称得知对应的接口。例如你的网页中想来上这样一个链接：

/Ordersheet
?action
=delete&&oid=12345

所以

IServiceProvider

的问题总结是：

－

在容器中包含很多重复的服务，不过他们作用的资源不同，操作也可能不同，

IServiceProvider

无法提供这个特性；

－

 IServiceProvider

还无法检索某个资源的特定接口的支持情况。

所以，

IResourceServiceProvider

诞生了。

![](/images/2006-10-24-Improvements-to-IServiceProvider-with-IResourceServiceProvider-3.gif)

新增的

resource

参数帮助定位不同的资源，以便获取不同的实现。通过这样的设计，你将解决上面的问题。
