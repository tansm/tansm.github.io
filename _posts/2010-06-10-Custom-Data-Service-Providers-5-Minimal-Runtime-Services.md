# 自定义Data Service Providers — （5）最小化的运行时服务

> 原文：https://www.cnblogs.com/tansm/archive/2010/06/10/DSP5.html

完整教程目录请参见：《**[自定义Data Service Providers — 简介](http://www.cnblogs.com/tansm/archive/2010/06/10/1755250.html)**》

在前面的教程中，我们已经实现了IServiceProvider和IDataServiceMetadataProvider，现在我们继续。

## 挂接IDataServiceQueryProvider实现

现在我们打算实现[IDataServiceQueryProvider](http://msdn.microsoft.com/en-us/library/system.data.services.providers.idataservicequeryprovider\(VS.100\).aspx)接口，在之前我们先重构一下之前实现的[IServiceProvider](http://msdn.microsoft.com/en-us/library/system.iserviceprovider.aspx)接口。

```csharp
public abstract class DSPDataService<T> :
DataService<T>, IServiceProvider
{
public IDataServiceMetadataProvider \_metadata;
public IDataServiceQueryProvider \_query;
public DSPDataService()
{
\_metadata = GetMetadataProvider(typeof(T));
\_query = GetQueryProvider(\_metadata);
}
public object GetService(Type serviceType)
{
if (serviceType == typeof(IDataServiceMetadataProvider))
return \_metadata;
else if (serviceType == typeof(IDataServiceQueryProvider))
return \_query;
else
return null;
}
public abstract IDataServiceMetadataProvider
GetMetadataProvider(Type dataSourceType);
public abstract IDataServiceQueryProvider
GetQueryProvider(IDataServiceMetadataProvider metadata);
}
```
注意：你需要在你自己的实现类中实现GetQueryProvider方法。

IDataServiceQueryProver实现可能需要IDataServiceMetadataProvider的实例，所以这里传入了这个参数。

```csharp
public class Sample : DSPDataService<object>
{
… 参见之前章节的代码 …
public override IDataServiceQueryProvider
GetQueryProvider(IDataServiceMetadataProvider metadata)
{
return new DSPQueryProvider<object>();
}
}
```
## 实现IDataServiceQueryProvider

如果你仅仅只是想看见服务契约和$metadata信息，那么实现CurrentDataSource属性就可以了，此接口的其他方法都是在调用查询返回ResourceSets时才会被调用。

```csharp
public class DSPQueryProvider<T> : IDataServiceQueryProvider
{
T\_currentDataSource;
public object CurrentDataSource
{
get {
return \_currentDataSource;
}
set {
\_currentDataSource= value as T;
}
}
public object GetOpenPropertyValue(
object target,
string propertyName)
{
throw new NotImplementedException();
}
public IEnumerable<KeyValuePair<string, object>>
GetOpenPropertyValues(object target)
{
throw new NotImplementedException();
}
public object GetPropertyValue(
object target,
ResourceProperty resourceProperty)
{
throw new NotImplementedException();
}
public IQueryable GetQueryRootForResourceSet(
ResourceSet resourceSet)
{
throw new NotImplementedException();
}
public ResourceType GetResourceType(object target)
{
throw new NotImplementedException();
}
public object InvokeServiceOperation(
ServiceOperation serviceOperation,
object\[\] parameters)
{
throw new NotImplementedException();
}
public bool IsNullPropagationRequired
{
get { throw new NotImplementedException(); }
}
}
```
好了，至少现在我们的自定义数据服务提供者能够跑起来了，虽然他仅能够提供元数据信息。

让我们在浏览器中键入：[http://localhost/sample.svc](http://localhost/sample.svc)，现在你可以看见下面的画面：

（译者注：在Visual Studio调试环境下，localhost后面需要有个动态端口）

![](/images/2010-06-10-Custom-Data-Service-Providers-5-Minimal-Runtime-Services-1.png)

如果你键入[http://localhost/sample.svc/$metadata](http://localhost/sample.svc/$metadata)的话，你应该看见这样的画面：

![](/images/2010-06-10-Custom-Data-Service-Providers-5-Minimal-Runtime-Services-2.png)

当然，我们没有实现查询，如果你要导航到Products资源集的话，会收到一个错误（地址是：[http://localhost/sample.svc/products](http://localhost/sample.svc/products)

![](/images/2010-06-10-Custom-Data-Service-Providers-5-Minimal-Runtime-Services-3.png)
