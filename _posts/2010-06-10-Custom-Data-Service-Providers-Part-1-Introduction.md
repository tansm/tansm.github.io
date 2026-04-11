---
title: "自定义Data Service Providers — （1）简介"
date: 2010-06-10 00:00:00 +0800
---

作者：AlexJ

翻译：谈少民

原文链接：[http://blogs.msdn.com/b/alexj/archive/2010/01/07/data-service-providers-getting-started.aspx](http://blogs.msdn.com/b/alexj/archive/2010/01/07/data-service-providers-getting-started.aspx)

# 简介

Data Services 建立于 Data Service Provider（数据服务提供者）之上，他负责Data Service与数据源之间的通讯。

Data Services内置了一些提供者，而且也允许你自定义数据提供者。

那么显而易见的问题是：

## 为什么要自定义数据提供者？

创建一个自定义的数据提供者十分有意义，你将获得巨大的收益，你可以：

l         在以下客户端程序中，查询和操作你的数据

n         WPF

n         WinForms

n         SilverLight

n         等等

l         在浏览器中直接查询和操控你的数据；

l         使用JavaScript或Jquery这样的框架程序查询和操控你的程序；

l         利用诸如[PowerPivot](http://www.powerpivot.com/)这样的数据分析工具查询你的数据；

l         等等……

但是在你准备自定义Data Service Provider之前，建议你先阅读以下内容，因为我们已经提供了一些内置的方案。

### Entity Framework（实体框架）

如果你计划使用Entity Framework构建数据访问层，在此基础上搭建你的Data Service，那么你无须自定义Data Service Provider。

Data Services已经内置了对Entity Framework 的数据提供者。

现在你可以非常简单的使用强类型的ObjectContext对象构建Data Service，就像这样：

***public class NorthwindDataService: ***

***DataService<NorthwindEntities>***

在上面的例子代码中，NorthwindEntities是你使用Entity Framework生成的强类型ObjectContext类。

### LINQ to SQL

如果你的Data Services使用LINQ to SQL作为数据访问层，那么你可以参考这个项目：

[ADO.NET Data Services IUpdateable implementation for Linq to Sql](http://code.msdn.microsoft.com/IUpdateableLinqToSql)

这个项目提供了例子代码，指导您实现一个强类型的DataContext类，此类实现了Iupdatable接口。这样你就能够使用Entity Framework相同的方式构建LINQ to SQL的数据提供者了。

***public class NorthwindDataService: ***

***DataService<NorthwindDataContext>***

### Reflection Provider（反射提供者）

如果你使用自定义类来提供数据源，且这些类包含一些返回类型是Iqueryable接口的属性，就像这样：

***public class MyDataSource ***

***{ ***

***    public IQueryable<Product> Products { get {…} } ***

***    public IQueryable<Categories> Categories { get {…} } ***

***}***

那么内建的反射提供者能够自动提供只读的数据服务，推断ResourceSets、类型和属性。

当然，你也可以通过实现IUpdatable接口来支持写入功能。

***public class MyDataSource: IUpdatable***

此特性就是基于上面在LINQ to SQL中提到的功能。

## 什么时候必须自定义数据提供者呢？

在通常的场景下，反射提供者是一个比较好的选择，但是，他也有一些限制：

1、 他必须是静态的，你的服务是固定的；

2、 必须有一个事实存在的CLR类来描述资源类型，很可能你没有这个类；

3、 你必须有一个Id属性或者有个{Type}Id 这样名称的属性作为你的键；

4、 你需要将CLR类中所有有关的属性都要公开出来；

5、 在流和分页控制上，你没有太多的定制能力；

6、 你无法获得一些高级功能，例如Open Types特性，此特性允许你[Open Properties](http://msdn.microsoft.com/en-us/library/system.data.services.providers.idataservicequeryprovider.getopenpropertyvalue(VS.100).aspx)上有更多的选择。

7、 一些细节你也很难定制，例如你不能够方便的记录请求，或者修改元数据以及重命名属性；

8、 等等……

如果这些问题是你所在乎的，那么你就必须自定义Data Service Provider了……

## 创建自定义数据提供者系列教程

在这个系列教程中，我们将展现大量的DSP(Data Service Provider)接口实现以及应用场景。

1、 [概述](http://www.cnblogs.com/tansm/archive/2010/06/10/DSP2.html)

2、 [IServiceProvider和DataSources 服务提供者和数据源;](http://www.cnblogs.com/tansm/archive/2010/06/10/DSP3.html)

3、 [IDataServiceMetadataProvider元数据提供者](http://www.cnblogs.com/tansm/archive/2010/06/10/DSP4.html)；

4、 [最小化的运行时服务；](http://www.cnblogs.com/tansm/archive/2010/06/10/DSP5.html)

5、 [查询；](http://www.cnblogs.com/tansm/archive/2010/06/10/DSP6.html)

6、 [交互式查询；](http://www.cnblogs.com/tansm/archive/2010/06/10/DSP7.html)

7、 [数据更新；](http://www.cnblogs.com/tansm/archive/2010/06/10/DSP8.html)

8、 [关系](http://www.cnblogs.com/tansm/archive/2010/06/10/DSP9.html)

9、 动态类型

未来将包含更多的教程

10、              ETags

11、              订阅支持；

12、              数据流

13、              高级分页功能

	posted on
2010-06-10 09:04
[编写人生](https://www.cnblogs.com/tansm)
阅读(2886)
评论(3)

[收藏](javascript:void(0))
[举报](https://report.cnblogs.com?targetLink=https%3A%2F%2Fwww.cnblogs.com%2Ftansm%2Farchive%2F2010%2F06%2F10%2F1755250.html&targetId=1755250&targetType=0)
