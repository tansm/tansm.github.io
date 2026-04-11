---
title: "自定义Data Service Providers — （2）概述"
date: 2010-06-10 00:00:00 +0800
---

完整教程目录请参见：《**[自定义Data Service Providers — 简介](http://www.cnblogs.com/tansm/archive/2010/06/10/1755250.html)**》

Data Services一个很酷的功能就是Provider模型。

任何一种数据源只要通过实现一些接口就可以支持[OData](http://odata.org/)数据服务，SharePoint 2010就实现了这些接口从而对外公开了其数据，那么你也可以这样做来公开你的Facebook,Twriter……

一旦你实现了这些接口，你就可以使用支持OData的客户端来查询你的数据，这些客户端包括PowerPivot、LINQ等等。

目前的Data Services内置提供了一些提供者程序，包括Entity Framework和使用反射访问对象的反射提供者。

在这个系列的教程中，我将一步一步的介绍如何自定义数据提供者。

## DSP接口

DSP包含5个重要的接口，他们是：

l         [IDataServiceMetadataProvider](http://msdn.microsoft.com/en-us/library/system.data.services.providers.idataservicemetadataprovider(VS.100).aspx)

通过实现此接口，对外公开可用的资源类型(ResourceType)，属性、关键字、嵌套属性和资源集(ResourceSets)；

l         [IDataServiceQueryProvider](http://msdn.microsoft.com/en-us/library/system.data.services.providers.idataservicequeryprovider(VS.100).aspx)

通过此接口完成所有真实的Get请求；

l         [IDataServiceUpdateProvider](http://msdn.microsoft.com/en-us/library/system.data.services.providers.idataserviceupdateprovider(VS.100).aspx)

如果你的数据源是可读写的，你还需要实现这个接口来支持诸如PUT、POST和DELETE的请求。

l         [IDataServicePagingProvider](http://msdn.microsoft.com/en-us/library/system.data.services.providers.idataservicepagingprovider(VS.100).aspx)

通过实现此接口你可以获得更细粒度的“[服务端分页](http://blogs.msdn.com/astoriateam/archive/2009/03/19/ado-net-data-services-v1-5-ctp1-server-driven-paging.aspx)”控制。

l         [IDataServiceStreamProvider](http://msdn.microsoft.com/en-us/library/system.data.services.providers.idataservicestreamprovider(VS.100).aspx)

如果还需要支持诸如多媒体的流输出，你还需要实现这个接口来支持流的输出。

所以你要实现最简单的只读数据服务，你必须实现[IDataServiceMetadataProvider](http://msdn.microsoft.com/en-us/library/system.data.services.providers.idataservicemetadataprovider(VS.100).aspx)和[IDataServiceQueryProvider](http://msdn.microsoft.com/en-us/library/system.data.services.providers.idataservicequeryprovider(VS.100).aspx)接口。

	posted on
2010-06-10 09:08
[编写人生](https://www.cnblogs.com/tansm)
阅读(877)
评论(0)

[收藏](javascript:void(0))
[举报](https://report.cnblogs.com?targetLink=https%3A%2F%2Fwww.cnblogs.com%2Ftansm%2Farchive%2F2010%2F06%2F10%2FDSP2.html&targetId=1755259&targetType=0)
