# 自定义Visual Studio IntelliTrace 智能跟踪

> 原文：https://www.cnblogs.com/tansm/archive/2012/06/14/CustomIntelliTrace.html

简介

我们知道Vistual Studio 2010提供了新的IntelliTrace智能追踪技术，他帮助我们不用单步调试就可快速的查看一段时间里发生的重要事件，关于智能跟踪可以参考《[使用 IntelliTrace 调试应用程序](http://msdn.microsoft.com/zh-cn/magazine/ee336126.aspx)》。但内置的事件往往还不够，我们希望扩展这些事件，本文就是介绍如何自定义扩展这些事件。

## 需求简介

在我们开发的一个Web项目中，客户端与服务端重要的一个交互就是传递“变更集”的Web请求，他是程序执行的关键事件。虽然智能追踪帮我们提供了Web请求的事件，但是他太多了，而且不能给我应用变更集事件关键的会话ID和xml数据，因此，我希望能够定制自己的“Apply ChangeSet”事件。

看起来定制后的效果应该是这样的：

![Image](/images/2012-06-14-Custom-Visual-Studio-IntelliTrace-1.png)

Visual Studio的Intellitrace面板能够看见我定制的“Apply ChangeSet”事件，除此之外，我还希望看见应用变更集时的相关数据：

![Image](/images/2012-06-14-Custom-Visual-Studio-IntelliTrace-2.png)

我的机器安装的是Vistual Studio 2011 beta，所以画面可能与Visual Studio 2010稍微不同。

## 定制追踪计划 collectionplan.xml

你需要知道的是，我们并不需要修改源代码，才能完成事件的记录，VS是通过一个collectionplan.xml描述追踪哪些方法，我们要改动这个文件来完成定制，他通常存在：

C:\Program Files (x86)\Microsoft Visual Studio 11.0\Common7\IDE\CommonExtensions\Microsoft\IntelliTrace\11.0.0\en

不同版本的Visual Studio其路径肯定不同，所以你需要自己搜索一下collectionplan.xml。我是直接修改这个文件，我没有找到不破坏这个文件另外建立文件的方法，如果你知道请回复我啊。

打开文件后，我们首先增加“分类”和“组件”两个数据，让我们先改动再看效果就很简单了。

![Image](/images/2012-06-14-Custom-Visual-Studio-IntelliTrace-3.png)

在Categories建立自己的分类，id和_locID老实说我不清楚区别，我照葫芦画瓢而已。

![Image](/images/2012-06-14-Custom-Visual-Studio-IntelliTrace-4.png)

在ModuleSpecifications中添加你希望追踪的dll，写文件名就可以了，不用关心版本，所在路径的问题。

![Image](/images/2012-06-14-Custom-Visual-Studio-IntelliTrace-5.png)

然后在DiagnosticEventSpecifications中添加你希望追踪的事件，他实际上就是定义你追踪哪个方法，以及你希望记录哪些变量。

简单的我就不介绍了，我也是照葫芦画瓢，我说明一下稍微复杂点的。

CatrgoryId和ModuleSpecificationId

CatrgoryId是分类的id，ModuleSpecificationId是前面我们填写的dll时对应的Id，不要搞错了。从这里我们也看出来，一个分类可以包含多个dll的事件，也可以一个dll发送不同分类的事件，完全由你规划。

MethodId:

我猜想是方法的签名，反正我严格按照规矩填写就是了，多个参数使用英文的逗号隔开，如果你没有返回值，可以填写: System.Void。

ShortDescription和LongDescription

顾名思义，一个是短描述和长描述，短描述显示在标题，长描述需要点击后才能看见。所以短描述要精简并显示关键数据。他支持变量的，就像我上面的{0}，和{1}，他指向DataQueries集合的索引。

DataQuery

这里你可以定义你希望记录的变量，第一个index的意思是你从哪里取值，0就是this，1及其以后就是各个参数。

如果取值的对象是一个简单类型，例如我这里取的session就是一个字符串，那么query属性就是空，如果取值的源是对象，你可以使用query获取其中的字段值，例如在SqlCommand中，他的长描述中输出了命令关联的连接字符串，像这样：

![Image](/images/2012-06-14-Custom-Visual-Studio-IntelliTrace-6.png)

显然他可以使用私有变量，并不断的层级访问。而且你不必小心处理_connection是null造成表达式失败。

至于如何获取返回值，(⊙o⊙)…我没有找到。

编程方式获取值

如果注意往下看其他的配置，好像还支持编程的方式获取值，例如：

![Image](/images/2012-06-14-Custom-Visual-Studio-IntelliTrace-7.png)

这个Microsoft.VistualStudio.DefaultDataQueries.dll也是用.net编写的程序，反编译你就能看见实现的方法，实现IProgrammableDataQuery接口即可。功能上当然要强一些，比如可以获取返回值，自己格式化数据。

## 在Visual Studio中查看效果

万事俱备，保存文件后你需要重新启动Visual Studio，先点击工具-》选项-》智能追踪。你可以看见我们自定义的分类。

![Image](/images/2012-06-14-Custom-Visual-Studio-IntelliTrace-8.png)

现在当你调试代码时，就可以在InterlliTrace中看见我们拦截的事件了。

调试愉快

![Image](/images/2012-06-14-Custom-Visual-Studio-IntelliTrace-9.png)
