# 自定义Data Service Providers — （4）IDataServiceMetadataProvider元数据提供者

> 原文：https://www.cnblogs.com/tansm/archive/2010/06/10/DSP4.html

完整教程目录请参见：《**[自定义Data Service Providers — 简介](http://www.cnblogs.com/tansm/archive/2010/06/10/1755250.html)**》

在这个章节我们将了解如何实现[IDataServiceMetadataProvider](http://msdn.microsoft.com/en-us/library/system.data.services.providers.idataservicemetadataprovider\(VS.100\).aspx)接口。

IDataServiceMetadataProvider对外提供你的数据源有哪些资源类型(ResourceType)和资源集(ResourceSet)。

那么你的数据源可能有哪些资源类型呢？

l         **他可能是完全静态的**，例如，在你的软件中或模型中，资源类型在软件设计时就已经是确定的，像Twriter、Stackoverflow这样的具体项目；

l         **他可能是强类型的，**例如你可能有个强类型类（表示你的数据源，使用T表示），通过他来反射获取有哪些属性，并在创建资源集时返回IQueryable<T>这样的结果，以及通过类型T来找到有多少资源类型(ResourceType)。Entity Framework就是这样方式。

l         **他可能完全是动态的，**例如你写了一个通用的DSP，只需要提供Database信息就可以对外提供元数据，在这种情况下，软件设计之初完全不知道最终运行时的类型，一切都是动态的。

灵活性还是很好的。

不论哪种方式，第一步你还是要建立一个实现IDataServiceMetadataProvider接口的类。

```csharp
public class DSPMetadata: IDataServiceMetadataProvider
```
这个接口包含6个方法和5个属性，听起来有点复杂，但是在我的第一个例子里，我只花了不到半个小时就搞定了。

```csharp
public abstract class DSPDataService<T>: DataService<T>
{
IDataServiceMetadataProvider \_metadata;
public DSPMetadata()
{
\_metadata = GetMetadataProvider(typeof(T));
}
public object GetService(Type serviceType)
{
if (serviceType == typeof(IDataServiceMetadataProvider))
{
return \_metadata;
}
…
}
public abstract IDataServiceMetadataProvider GetMetadataProvider(Type dataSourceType);
}
```
在这段代码中，你可以看见我定义了一个抽象方法用来获取元数据，你必须在派生类中实现它。

在实现这个接口之前，你还得了解……

## 创建元数据

也许你已经注意到这个接口上包含这些概念：[ResourceTypes](http://msdn.microsoft.com/en-us/library/system.data.services.providers.resourcetype\(VS.100\).aspx)、[ResourceProperties](http://msdn.microsoft.com/en-us/library/system.data.services.providers.resourceproperty\(VS.100\).aspx)、[ServiceOperations](http://msdn.microsoft.com/en-us/library/system.data.services.providers.serviceoperation\(VS.100\).aspx)、[ResourceSets](http://msdn.microsoft.com/en-us/library/system.data.services.providers.resourceset\(VS.100\).aspx)和[ResourceAssociationSets](http://msdn.microsoft.com/en-us/library/system.data.services.providers.resourceassociationset\(VS.100\).aspx)。

这些类公开定义在System.Data.Services组装件下，这样你就能够使用这些类来描述你的数据模型，注意这些类仅仅用来描述你的模型信息。

试想一下你有一个CLR类：

```csharp
public class Product
{
public int ProdKey {get;set;}
public string Name {get;set;}
public Decimal Price {get;set;}
public Decimal Cost {get;set;}
}
```
现在你需要为这个Product类创建一个ResourceType来描述它，你该怎样做呢

```csharp
var productType = new ResourceType(
typeof(Product), // 资源在最终实例化时的CLR类型
ResourceTypeKind.EntityType, // 例如是实体、复杂类型等
null, // 此类型的基类
"Namespace", // 命名空间
"Product", // 名称
false // 是否是抽象类型
);
var prodKey = new ResourceProperty(
"ProdKey",
ResourcePropertyKind.Key |
ResourcePropertyKind.Primitive,
ResourceType.GetPrimitiveResourceType(typeof(int))
);
var prodName = new ResourceProperty(
"Name",
ResourcePropertyKind.Primitive,
ResourceType.GetPrimitiveResourceType(typeof(string))
);
var prodPrice = new ResourceProperty(
"Price",
ResourcePropertyKind.Primitive,
ResourceType.GetPrimitiveResourceType(typeof(Decimal))
);
productType.AddProperty(prodKey);
productType.AddProperty(prodName);
productType.AddProperty(prodPrice);
```
注意：我们并没有提供Cost（成本）属性，因为我们假设他是敏感信息。

下一步，我们将使用这个资源类型来创建一个叫Products的资源集，非常简单：

```csharp
var productsSet = new ResourceSet("Products", productType);
```
正如你所看到的，构建一个元数据并不难。

## 冻结你的元数据

最后一个步骤就是要冻结你的元数据信息：

```csharp
productsSet.SetReadonly();
productType.SetReadonly();
```
这句话的意思是：不可以修改元数据了，至少在当前请求下不可以。

注意：Data Services在每次请求前，都将查询IDataServiceMetadataProvider实现程序其元数据是否已冻结，如果未冻结，那么每次请求都将要求重建元数据。

## 通过IDataServiceMetadataProvider暴露元数据

一旦建立好元数据，你就需要暴露这些元数据。

如果你不打算支持继承、关系和服务操作的话，那么实现就比较简单，就像这样：

```csharp
public class DSPMetadataProvider
{
private Dictionary<string, ResourceType> \_resourceTypes
= new Dictionary<string, ResourceType>();
private Dictionary<string, ResourceSet> \_resourceSets
= new Dictionary<string, ResourceSet>();
public DSPMetadataProvider(){}
public void AddResourceType(ResourceType type)
{
type.SetReadOnly();
\_resourceTypes.Add(type.FullName, type);
}
public void AddResourceSet(ResourceSet set)
{
set.SetReadOnly();
\_resourceSets.Add(set.Name, set);
}
public string ContainerName
{
get { return "Container"; }
}
public string ContainerNamespace
{
get { return "Namespace"; }
}
public IEnumerable<ResourceType> GetDerivedTypes(
ResourceType resourceType
)
{
//这里我们暂不实现继承
yield break;
}
public ResourceAssociationSet GetResourceAssociationSet(
ResourceSet resourceSet,
ResourceType resourceType,
ResourceProperty resourceProperty)
{
throw new NotImplementedException("不支持关系");
}
public bool HasDerivedTypes(ResourceType resourceType)
{
// 这里我们暂不实现继承
return false;
}
public IEnumerable<ResourceSet> ResourceSets
{
get { return this.resourceSets.Values; }
}
public IEnumerable<ServiceOperation> ServiceOperations
{
//这里我们暂不实现服务操作
get { yield break; }
}
public bool TryResolveResourceSet(
string name,
out ResourceSet resourceSet)
{
return resourceSets.TryGetValue(name, out resourceSet);
}
public bool TryResolveResourceType(
string name,
out ResourceType resourceType)
{
return resourceTypes.TryGetValue(name, out resourceType);
}
public bool TryResolveServiceOperation(
string name,
out ServiceOperation serviceOperation)
{
//这里我们暂不实现服务操作
serviceOperation = null;
return false;
}
public IEnumerable<ResourceType> Types
{
get { return this.resourceTypes.Values; }
}
}
```
下面是我们之前提到的GetMetadataProvider方法的实现例子：

```csharp
public override IDataServiceMetadataProvider GetMetadataProvider(Type dataSourceType)
{
DSPMetadataProvider metadata = new DSPMetadataProvider();
var productType = new ResourceType(
typeof(Product), // CLR type backing this Resource
ResourceTypeKind.EntityType, // Entity, ComplexType etc
null, // BaseType
"Namespace", // Namespace
"Product", // Name
false // Abstract?
);
var prodKey = new ResourceProperty(
"ProdKey",
ResourcePropertyKind.Key |
ResourcePropertyKind.Primitive,
ResourceType.GetPrimitiveResourceType(typeof(int))
);
var prodName = new ResourceProperty(
"Name",
ResourcePropertyKind.Primitive,
ResourceType.GetPrimitiveResourceType(typeof(string))
);
var prodPrice = new ResourceProperty(
"Price",
ResourcePropertyKind.Primitive,
ResourceType.GetPrimitiveResourceType(typeof(Decimal))
);
productType.AddProperty(prodKey);
productType.AddProperty(prodName);
productType.AddProperty(prodPrice);
metadata.AddResourceType(productType);
metadata.AddResourceSet(
new ResourceSet("Products", productType)
);
return metadata;
}
```
注意：我们在添加ResourceType和ResourceSet时，已经自动调用了冻结(SetReadOnly)方法了。

当然，如果你想支持继承、关系和服务操作，那么做起来就比较复杂，但是你通过LINQ to Objects做就比较容易了。

## 性能考虑

你可能已经注意到，在这个接口中包含两组方法和属性，一组是返回枚举器（Enumeration），一组是使用Try方法尝试获取信息，这种设计主要是基于性能的考虑。

大多数的服务请求都会调用TryResolveXXX这样的方法，这样你就可以延迟创建这些元数据信息。

有些方法需要谨慎调用，例如：当调用get$metadata或者根文档时。（译者注：啥意思，没理解）。

现在我们已经将自定义的DSP实现挂接到DataService上，并且已经实现了IDataServiceMetadataProvider接口，下一步，我们将实现IDataServiceQueryProvider接口来实现只读的数据查询。
