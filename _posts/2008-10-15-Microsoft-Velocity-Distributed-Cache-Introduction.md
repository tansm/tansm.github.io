---
title: "Microsoft 分布式缓存 Velocity 使用简介"
date: 2008-10-15 00:00:00 +0800
---

# Microsoft 分布式缓存 Velocity 使用简介

（部分内容翻译自Microsoft的文章）

Velocity是Microsoft提供的分布式缓存技术解决方案，你可以使用任何一种.NET语言来访问他的API，他的主要资源包括：

Blog：[http://blogs.msdn.com/velocity/](http://blogs.msdn.com/velocity/)

论坛：[http://forums.microsoft.com/MSDN/ShowForum.aspx?ForumID=2142&SiteID=1](http://forums.microsoft.com/MSDN/ShowForum.aspx?ForumID=2142&SiteID=1)

下载：[http://www.microsoft.com/downloads/details.aspx?FamilyId=B24C3708-EEFF-4055-A867-19B5851E7CD2](http://www.microsoft.com/downloads/details.aspx?FamilyId=B24C3708-EEFF-4055-A867-19B5851E7CD2)

例子：[http://code.msdn.microsoft.com/velocity](http://code.msdn.microsoft.com/velocity)

## 准备

首先在上面提到的官方地址下载安装程序，运行安装程序并配置你的服务器。

安装后可以看出，Velocity支持集群模式的缓存服务器，他以服务的寄生在Windows中，并通过TCP与客户端交互。

在编写和测试下面的程序之前，请检查Windows的服务中Microsoft project code named “Velocity”已经启动。

## 基本测试程序

首先在Visual Studio 2008中建立一个解决方案（注意选择目标是.NET 3.5），然后复制

CacheBaseLibrary.dll

ClientLibrary.dll

FabricCommon.dll

CASBase.dll,

CASClient.dll

这5个文件到你的运行目录，当然你实际上只需要引用CacheBaseLibrary.dll和ClientLibrary.dll。

第二步，你还需要编写配置文件，打开app.config并填写诸如以下的配置

```text

<?
```

注意修改服务器的地址，我在试验时，使用计算机名称时怎么也连接不上(防火墙、服务都检查了，哎)，所以只好使用localhost了。

好了，可以写程序了，首先需要using System.Data.Caching;然后在程序中先来这句：

```text

static
```

后面的程序都是使用这个Cache对象测试的。

我为了测试其对私有对象类型的缓存我自己编写了一个产品类，很简单的。

```text

[Serializable]
```

首先需要将数据存放到缓存中

```text

//
```

在这个地方我使用了put方法，没有使用add方法，根据ms的帮助说明，put方法会在缓存如果包含此键时替换原始项目，当然如果没有则新增，而add在缓存中包含此键时播发异常。所以在并发环境下我更愿意使用put方法。

取数据也比较方便，

//获取数据

Product p2 = (Product)cache.Get(p1.ProductId);

当你的数据发生改变后，最简单的方法是从缓存中删除掉，而不是更新缓存。

cache.Remove(p1.ProductId);

现在你已经会使用缓存服务器做基本的功能了，现在学习一下稍微复杂的功能：分区。

## 分区

在上面的例子中，我们直接使用ProductID作为键，你可能会想到，在实际部署中可能只有一台缓存服务器，而我的应用服务器允许多个帐套，这样，就有可能帐套1的产品p1和帐套2的产品p2产生冲突，因为他们共享了相同的键。

解决方法就是建立不同的分区存储，事实上，上面的代码中，存储在默认的分区(null)中。好，让我们看看代码

```text

//
```

查看你的代码你会发现他们的确隔离了。

聪明的你可能已经想到，如果你的表使用int作为主键，那么即使使用帐套作为分区，各个表之间任然会键冲突。我的想法是：帐套名+实体名（表名）作为分区键，这样做的好处是，当你Reset某个表的所有数据时，你清理缓存也比较方便，只要RemoveRegion了。如果你使用帐套名作为分区，表+主键 作为缓存键，那么清理缓存可能不方便。

我在测试时发现一个麻烦的问题，调用CreateRegion或RemoveRegion方法很可能出错，因为当我上次测试失败后，缓存服务器并没有关闭，也就是说上次测试建立的Region可能存在也可能不存在，糟糕的是API中并没有看见任何检测Region是否存在或者自动建立Region的方法。

如果你认为最终用户那边有个不错的IT管理员，也可以考虑建立多个CacheName，修改ClusterConfig.xml，可是我没有成功，我猜的，嚯嚯。

## Tag

Tag我翻译成标签，Velocity中允许你为加入缓存的数据添加标签，标签可以是多个，用过Gmail吗？他的Label有异曲同工的意思。我努力寻找使用的案例，可怎么也找不到一个贴切一点的，只好让大家看看怎么使用算了。

```text

p1
```

当然，如果你只检索一个标签，使用GetByTag更好些。

result = cache.GetByTag("AccountSet1", new Tag("b"));

还有GetAnyMatchingTags方法，看方法名就知道功能了。

我试图根据Tag同时删除一组数据，可惜没有找到这样的方法，不知道是不是他们忘记了， J

在Velocity还有很多好玩的东西，可是我不能试验成功，诸如版本功能，锁定等，同时他还提供了IIS的会话提供者，这样你就可以使用另外一台机器作为会话服务器了。

[点击这里下载测试程序](https://files.cnblogs.com/tansm/CacheTest.rar)
