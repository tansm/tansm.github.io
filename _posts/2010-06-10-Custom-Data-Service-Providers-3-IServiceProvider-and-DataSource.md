# 自定义Data Service Providers — （3）IServiceProvider和DataSources 服务提供者和数据源

> 原文：https://www.cnblogs.com/tansm/archive/2010/06/10/DSP3.html

完整教程目录请参见：《**[自定义Data Service Providers — 简介](http://www.cnblogs.com/tansm/archive/2010/06/10/1755250.html)**》

要创建只读的DSP你必须实现两个接口，[IDataServiceMetadataProvider](http://msdn.microsoft.com/en-us/library/system.data.services.providers.idataservicemetadataprovider\(VS.100\).aspx)用于提供元数据定义，[IDataServiceQueryProvider](http://msdn.microsoft.com/en-us/library/system.data.services.providers.idataservicequeryprovider\(VS.100\).aspx)用于处理查询。

但是在实现此接口之前，你还需要先实现[IServiceProvider](http://msdn.microsoft.com/en-us/library/system.iserviceprovider.aspx)接口来让Data Services能够找到你的接口实现。

## IServiceProvider

Data Services通过[IServiceProvider](http://msdn.microsoft.com/en-us/library/system.iserviceprovider.aspx)接口来查找你的实现。

最佳的途径是创建派生自DataService<T>类并实现IServiceProvider接口，例如：

```csharp
public class DSPDataService<T>: DataService<T>, IServiceProvider
{
public object GetService(Type serviceType)
{
if (serviceType == typeof(IDataServiceMetadataProvider))
{
return …
}
else if (serviceType == typeof(IDataServiceQueryProvider))
{
return …
}
else
{
return null;
}
}
}
```
当然，具体的代码要看你最终的实现，我们将在稍后介绍。

在你的Web应用程序或[WCF应用程序](http://blogs.msdn.com/alexj/archive/2009/12/11/tip-48-how-to-host-a-data-service-in-wcf.aspx)中，你需要添加一些代码以便这些程序可以了解你的DataService能够提供的访问入口（译者注：例如告诉这些程序你可以提供Customers的只读访问），在下面的列子中还演示了不从DataService<T>派生的情况。

```csharp
public class DemoDSPDataService : DSPDataService<DSPContext>
{
// This method is called only once to initialize
// service-wide policies.
public static void InitializeService(
DataServiceConfiguration config
)
{
// TODO: replace this sample configuration with
// real configuration
config.SetEntitySetAccessRule("\*", EntitySetRights.AllRead);
config.DataServiceBehavior.MaxProtocolVersion =
DataServiceProtocolVersion.V2;
config.DataServiceBehavior.AcceptProjectionRequests = true;
}
}
```
到现在，我们已经将我们的自定义DSP宿主到Data Service上了。

酷！

当然，我们还需要一些工作让他能够真正工作起来（译者注：哈哈，路还远呢）。

在这之前，我们还需要处理一些其他事宜。

## 数据从何而来

如果你仔细看了上面的代码：

```csharp
public class DemoDSPDataService : DSPDataService<DSPContext>
```
你可能已经注意到:DSPContext是个啥？

实际上，他就是代表你的数据源，通过它来识别你的数据源。

作为参考的例子，在Entity Framework的DSP实现中，这个类就是派生自[ObjectContext](http://msdn.microsoft.com/en-us/library/system.data.objects.objectcontext.aspx)的类。

备注：你没有必要也这么做，因为Entity Framework已经内置了此功能。

如果你的DSP程序支持通过强类型类来定义数据源，就像Entity Framework那样，那么我建议你创建一个使用强类型数据源作为泛型参数的类，例如:

```csharp
public class DSPDataService<T>: DataService<T>, IServiceProvider
where T: MyDataSource
```
如果你的数据源是固定的，那么你就不需要定义这个泛型基类，像下面这样就可以了：

```csharp
public class DSPDataService: DataService<MyDataSource>,
IServiceProvider
```
正如你上面所了解的，如何定义取决于你的需求。
