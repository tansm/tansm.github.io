# 自定义Data Service Providers —（6）查询

> 原文：https://www.cnblogs.com/tansm/archive/2010/06/10/DSP6.html

完整教程目录请参见：《**[自定义Data Service Providers — 简介](http://www.cnblogs.com/tansm/archive/2010/06/10/1755250.html)**》

在上面的章节中，我们挂接了自定义的服务提供者，并且实现了IDataServiceQueryProvider接口，虽然他只能用来查询元数据和服务契约。

在这一部分，我们让查询也可以运行起来，要做到这点我们必须知道数据从哪里来：数据源，以及我们必须实现[IDataServiceQueryProvider](http://msdn.microsoft.com/en-us/library/system.data.services.providers.idataservicequeryprovider\(VS.100\).aspx)接口。

## 创建数据源

那么我们的Product数据在那么找到呢？

我们先假设数据是放在内存里面，这样实现比较简单。现在我们第一步先编写一个基类：

```csharp
public abstract class DSPContext
{
public abstract IQueryable GetQueryable(ResourceSet set);
}
```
然后我们设计一个仅能处理Products数据的强类型类，看起来像这样：

```csharp
public class ProductsContext: DSPContext
{
private List<Product> \_products = new List<Product>();
public override IQueryable GetQueryable(ResourceSet set)
{
if (set.Name == "Products") return Products.AsQueryable();
throw new NotSupportedException(
string.Format("{0} 未找到", set.Name)
);
}
public List<Product> Products
{
get {
return \_products;
}
}
}
```
到目前还是蛮简单的，你应该明白其ResourceSet背后的数据源就是一个简单的List。

这里需要注意的是，由于我们的数据放在内存里，所以我们这里使用了LINQ to Objects技术提供的.AsQueryable()方法实现IQueryable的返回值。

接下来，我们需要修改我们的Sample服务类，他使用ProductsContext作为数据源。

```csharp
public class Sample : DSPDataService<ProductsContext>
{
```
我们还需要重载这个类的CreateDataSource方法，以便预先加入一些测试数据进去。

```csharp
protected override ProductsContext CreateDataSource()
{
ProductsContext context = new ProductsContext();
context.Products.Add(
new Product {
ProdKey = 1,
Name = "Bovril",
Cost = 4.35M,
Price = 6.49M
});
context.Products.Add(
new Product {
ProdKey = 2,
Name = "Marmite",
Cost = 4.97M,
Price = 7.21M
});
return context;
}
```
最后一步，我们需要完成GetQueryProvider方法（需要参见前面教程中Sample类的代码，那个时候我们填写的是Object，而不是下面的ProductsContext）。

```csharp
public override IDataServiceQueryProvider GetQueryProvider(
IDataServiceMetadataProvider metadata)
{
return new DSPQueryProvider<ProductsContext>(metadata);
}
```
## 为强类型数据实现IDataServiceQueryProvider

现在，我们需要重新编写上个教程章节中编写的DSPQueryProvider<T>类，当时那个类很多功能都没有实现。

而我们现在需要实现一个对内存中强类型类数据的查询提供者，这里所说的“强类型类”是指：使用ResourceType描述类型信息并且其描述的属性都能够找到真实的CLR属性与之对应。

代码如下：

```csharp
public class DSPQueryProvider<T> : IDataServiceQueryProvider where T : DSPContext
{
T \_currentDataSource;
IDataServiceMetadataProvider \_metadata;
public DSPQueryProvider(IDataServiceMetadataProvider metadata)
{
\_metadata = metadata;
}
public object CurrentDataSource
{
get { return \_currentDataSource;}
set { \_currentDataSource = value as T; }
}
public IQueryable GetQueryRootForResourceSet(
ResourceSet resourceSet)
{
return \_currentDataSource.GetQueryable(resourceSet);
}
public ResourceType GetResourceType(object target)
{
Type type = target.GetType();
return \_metadata.Types
.Single(t => t.InstanceType == type);
}
public bool IsNullPropagationRequired
{
get {return true; }
}
public object GetOpenPropertyValue(
object target,
string propertyName)
{
throw new NotImplementedException();
}
public IEnumerable<KeyValuePair<string, object>> GetOpenPropertyValues(object target)
{
throw new NotImplementedException();
}
public object GetPropertyValue(
object target,
ResourceProperty resourceProperty)
{
throw new NotImplementedException();
}
public object InvokeServiceOperation(
ServiceOperation serviceOperation,
object\[\] parameters)
{
throw new NotImplementedException();
}
}
```
## 实现说明

在上面的实现中，CurrentDataSource内部使用\_currentDataSource变量记录，其最终运行时指向ProductsContext的实例。

在下面的GetQueryRootForResourceSet方法中，我们使用了\_currentDataSource.GetQueryable()方法返回特定ResourcceSet（资源集）的查询对象(IQueryable)。

我们还实现了GetResourceType(object)方法，他首先获取Target的CLR类型，然后在IDataServiceMetadataProvider.Types中使用InstanceType查找与之匹配的类型。

最后，IsNullPropagationRequired属性返回了true，因为我们必须让Data Services处理LINQ to Objects无法处理的null。

### 什么是NullPropagation

让我们来看看一个LINQ的查询例子：

```csharp
var results = from x in queryRoot
select x.Customer.Name;
```
在这个例子里，如果x.Customer是null值，那么程序运行时将抛出NullReferenceException异常。

可以在上面的IsNullPropagationRequired属性实现中返回true，这样Data Services就会检查查询的数据是否是null从而防止出错。

注意：诸如LINQ to SQL和LINQ to Entity 这样的IQueryables数据库查询实现，是支持null值传播的，无须返回true。

## 哪些是未实现的

由于我们的Data Service Provider是针对强类型实体实现的，所以Data Services永远不会调用GetPropertyValue方法，因为默认情况下系统直接从实例的属性上读取。

译者注：可以查看一下ResourceProperty类包含CanReflectOnInstanceTypeProperty属性，默认情况下此属性返回true，表示直接使用反射获取值（实际上使用了Expression.Property加快了访问速度）。

在当前实现的模型中，我们没有支持服务操作(ServiceOperation)、开放类型(OpenType)和开放属性(OpenProperty)，所以我们都在对应的方法中抛出了NotImplemented异常。

**总结**

最终我们完成了只读方式的数据查询功能，键入：[http://localhost/simple.svc/products](http://localhost/simple.svc/products)看起来像这样:

![](/images/2010-06-10-Custom-Data-Service-Providers-6-Query-1.png)

我们还有很长的路要走，在后面的教程中，我们将逐步增加一些功能支持，包括数据更新、关系、服务操作、流、分页和开放类型，以及展现没有对应的CLR类型的情况。
