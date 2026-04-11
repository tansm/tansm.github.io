# 自定义Data Service Providers — （8）数据更新

> 原文：https://www.cnblogs.com/tansm/archive/2010/06/10/DSP8.html

完整教程目录请参见：《**[自定义Data Service Providers — 简介](http://www.cnblogs.com/tansm/archive/2010/06/10/1755250.html)**》

在前面的所有章节中，我们创建了使用内存存放数据的只读提供者，现在我们打算添加数据更新的功能。

要做到这点，必须实现[IDataServiceUpdateProvider](http://msdn.microsoft.com/en-us/library/system.data.services.providers.idataserviceupdateprovider\(VS.100\).aspx)接口。

但首先我们还需要了解……

## 批量处理

[IDataServiceUpdateProvider](http://msdn.microsoft.com/en-us/library/system.data.services.providers.idataserviceupdateprovider\(VS.100\).aspx)接口设计中支持批量特性，这就允许在一个事务中一次更新很多的资源。

这个接口很底层——一个API调用需要更新每一个独立的属性，甚至还要更新一个单独的资源，这个资源调用可能导致在一个原子级API调用批处理，如果这个API调用失败，我们需要回滚/放弃所有之前干过的工作。

换句话说，你可以在SaveChangeds()方法调用前指导IDataServiceUpdateProvider.SetValue(..)方法总共调用了多少次。

这似乎看起来比较简单，但是对接口的实现却影响很大。

在某个方法实现上，你将不可以立即应用请求的改变，而是记录所发生的事情并在最后一次性的提交所有操作。

如果你的数据存放在数据库中，那么数据库系统会自动的在事务中记录所有的命令操作，比如Entity Framewok就是这样的例子。

很不幸，在我之前的建立的例子里，我使用了内存中的对象存放数据，所以我们需要记录在SaveChangeds()前所发生的一切。

这就是为什么在我下面的实现中，使用Actions对象记录操作的原因。让我们来看看实现……

## 建造自己的DSPContext 更新策略

实现IDataServiceUpdateProvider的一个先决条件是数据源必须是可更新的。（译者注：有些废话）。

你可以有很多种实现方式，所以我在DSPContext上定义了一组抽象方法，通过这些方法我们可以实现创建资源(Create)、添加资源(Add)、删除资源(Delete)和提交(Save)。

```csharp
public abstract object CreateResource(ResourceType resourceType);
public abstract void AddResource(ResourceType resourceType,
object resource);
public abstract void DeleteResource(object resource);
public abstract void SaveChanges();
```
然后在我们的派生类ProductsContext上实现这些方法：

```csharp
public override object CreateResource(ResourceType resourceType)
{
if (resourceType.InstanceType == typeof(Product))
{
return new Product();
}
throw new NotSupportedException(
string.Format("{0} not found", resourceType.FullName)
);
}
public override void AddResource(
ResourceType resourceType,
object resource)
{
if (resourceType.InstanceType == typeof(Product))
{
Product p = resource as Product;
if (p != null){
Products.Add(p);
return;
}
}
throw new NotSupportedException("Type not found");
}
public override void DeleteResource(
object resource)
{
if (resource.GetType() == typeof(Product))
{
Products.Remove(resource as Product);
return;
}
throw new NotSupportedException("Type not found");
}
public override void SaveChanges()
{
var prodKey = Products.Max(p => p.ProdKey);
foreach (var prod in Products.Where(p => p.ProdKey == 0))
prod.ProdKey = ++prodKey;
}
```
看起来是不是很简单？在这里SaveChanges方法中，我们通过获取最大的编号并为新建的产品（ProdKey == 0）分配了编号来模拟保存到数据库。

该准备的都准备了，继续……

## 实现IDataServiceUpdateProvider

我们的实现是放在DSPUpdateProvider<T>上，代码如下：

```csharp
public class DSPUpdateProvider<T>: IDataServiceUpdateProvider
where T: DSPContext
```
在这个类中，我定义了一个泛型参数T来描述数据源。

IDataServiceUpdateProvider接口没有任何方法能够提供DSPContext实例，所以我需要定义一个构造函数接收IDataServiceMetadataProvider实例，通过它以便最终能够获取DSPContext。

还记得前面教程中关于IServiceProvider的实现吗？我们需要稍稍改变一下代码。

所以最终DSPUpdateProvider构造函数中，接收元数据和查询提供者两个参数。

```csharp
public DSPUpdateProvider(
IDataServiceMetadataProvider metadata,
DSPQueryProvider<T> query)
{
\_metadata = metadata;
\_query = query;
\_actions = new List<Action>();
}
```
这里的\_actions用来存放在IDataServiceUpdateProvider.SaveChanges()方法执行前所有的调用过的操作列表。

当然，你应该记得我们并不“真的”将数据更新到数据源。

### 获取数据源

下一步，我们需要能够获取数据源对象，例如派生自DSPContext的对象，代码看起来像这样：

```csharp
public T GetContext()
{
return (\_query.CurrentDataSource as T);
}
```
他是从IDataServiceQueryProvider中的CurrentDataSource属性上获取到数据源，并准换为T，当然，数据源必须是派生自DSPContext。

### 创建资源

现在，Data Services可能发生一个插入操作（例如POST操作），为支持此操作，我们需要在后台创建一个资源(Resource)，他是通过CreateResource方法实现的。

```csharp
public object CreateResource(
string containerName,
string fullTypeName)
{
ResourceType type = null;
if (\_metadata.TryResolveResourceType(fullTypeName, out type))
{
var context = GetContext();
var resource = context.CreateResource(type);
\_actions.Add(
() => context.AddResource(type, resource)
);
return resource;
}
throw new Exception(
string.Format("Type {0} not found", fullTypeName)
);
}
```
这这段代码中，我尝试创建ResourceType对应的CLR实例。

注意：我们虽然创建了资源(Resource)，但是我们并没有立即加入到我们的数据源中，而是将请求转换成一个委托并加入到委托列表中，供后面的程序使用。

### 获取资源

当Data Services层需要更新或删除某个资源（Resource）时，首先需要获取他，这个时候就会用到IDataServiceUpdateProvider.GetResource(…)方法。

Data Services传递一个IQueryable实例，此方法将返回结果中对应的资源对象Resource。

你可能会问：为什么DataServices不自己获取最终的资源对象(Resource)？

这是因为，这个方法提供了一种扩展能力，允许你返回一个代表真实资源的对象，而不是真实的资源（又名 proxy 代理）

例如，你可以使用这个“代理”记录数据的改变并在调用SaveChanges()方法时批量的提交操作。

但是在我们目前的实现中，我们仅仅简单的直接返回资源对象(Resource)。

当然，在这个实现中我们还检查返回的资源只能有一个且只有一个，而且必须是已知的ResourceType。

```csharp
public object GetResource(IQueryable query, string fullTypeName)
{
var enumerator = query.GetEnumerator();
if (!enumerator.MoveNext())
throw new Exception("没有找到任何查询结果。");
var resource = enumerator.Current;
if (enumerator.MoveNext())
throw new Exception("查询结果只能有一条");
if (fullTypeName != null)
{
ResourceType type = null;
if (!\_metadata.TryResolveResourceType(
fullTypeName, out type))
throw new Exception("指定的资源类型未找到");
if (!type.InstanceType.IsAssignableFrom(resource.GetType()))
throw new Exception("意外的资源类型");
}
return resource;
}
```
与之密切相关的是IDataServiceUpdateProvider的ResolveResource(…)方法。

如果我们在前面的GetResource(..)方法实现中，返回了代理而不是真实的资源，那么就需要通过这个方法再还原成真实的资源实例。

因为我之前的实现没有使用代理，所以我现在的方法实现很简单：

```csharp
public object ResolveResource(object resource)
{
return resource;
}
```
### 更新属性值

一旦Data Services获取到资源（或者资源的代理），他就有可能开始修改它，每次修改都会调用IDataServicesUpdateProvider的SetValues(…)方法。

在上面的说明中，我们已经了解：不可以立即更新属性的值而是将记录它的操作。

```csharp
public void SetValue(
object targetResource,
string propertyName,
object propertyValue)
{
// TODO: 在这里添加一些断言!!!
\_actions.Add(
() => ReallySetValue(
targetResource,
propertyName,
propertyValue)
);
}
public void ReallySetValue(
object targetResource,
string propertyName,
object propertyValue)
{
targetResource
.GetType()
.GetProperties()
.Single(p => p.Name == propertyName)
.GetSetMethod()
.Invoke(targetResource, new\[\] { propertyValue });
}
```
正如你所看见的，在ReallySetValue（抱歉，我没有想到一个更好的名字）方法中，我们使用了反射来设置属性的值。

你也许为了提高性能而不使用上面的反射代码，但是你得想想这是否值得，因为在序列化/反序列化、网络传输等“性能大户”面前，这点反射消耗相对来说微不足道。

### 获取属性值

偶尔在更新数据时，也需要获取属性的值，这将调用IDataServiceUpdateProvider的GetValue(…)方法。由于GetValue方法不会产生副作用，所以这里就直接的使用反射获取值。

```csharp
public object GetValue(object targetResource, string propertyName)
{
var value = targetResource
.GetType()
.GetProperties()
.Single(p => p.Name == propertyName)
.GetGetMethod()
.Invoke(targetResource, new object\[\] { });
return value;
}
```
### 删除资源

现在我们需要处理删除功能(Delete)，这段代码浅显易懂我就不解释了。

```csharp
public void DeleteResource(object targetResource)
{
\_actions.Add(() =>
GetContext().DeleteResource(targetResource)
);
}
```
### 重置资源

一般情况下，我们并不需要重置一个资源，但是如果客户端发起一个请求，例如：

```csharp
SaveChanges(SaveChangesOptions.ReplaceOnUpdate);
```
如果发生此请求，Data Services将会调用IDataServiceUpdateProvider的ResetResouce(..)来重置除主键之外的所有属性。

这是我的实现

```csharp
public object ResetResource(object resource)
{
\_actions.Add(() => ReallyResetResource(resource));
return resource;
}
```
具体实现代码如下（译者注：实现的是有点囧）：

```csharp
public void ReallyResetResource(object resource)
{
// 创建一个空的资源实例
var clrType = resource.GetType();
ResourceType resourceType =
\_metadata.Types.Single(t => t.InstanceType == clrType);
var resetTemplate = GetContext().CreateResource(resourceType);
// 从空的资源实例中复制所有属性（不包括主键）
foreach (var prop in resourceType
.Properties
.Where(p => (p.Kind & ResourcePropertyKind.Key)
!= ResourcePropertyKind.Key))
{
// 你可以为了性能缓存这些结果
var clrProp = clrType
.GetProperties()
.Single(p => p.Name == prop.Name);
var defaultPropValue = clrProp
.GetGetMethod()
.Invoke(resetTemplate, new object\[\] { });
clrProp
.GetSetMethod()
.Invoke(resource, new object\[\] { defaultPropValue });
}
}
```
上面的代码是利用“空”的实体资源，将其默认值复制到要重置的资源上(不包括主键的属性)。

### 保存

最后，我们要按计划完成最后一个步骤，一旦调用了IDataServiceUpdateProvider的SaveChanges方法，我们就会顺序的执行所有的步骤。

```csharp
public void SaveChanges()
{
foreach (var a in \_actions)
a();
GetContext().SaveChanges();
}
```
我们只是按顺序调用了actions队列中的所有方法，然后再调用了DSPContext的SaveChanges方法，如果你还记得的话，那个方法仅仅模拟了保存（为新增的实体更新了主键）。

如果一切顺利，ClearChanges()方法将被调用，这个实现更简单：

```csharp
public void ClearChanges()
{
\_actions.Clear();
}
```
好了，我们终于实现IDataServiceUpdateProvider接口了。

\\(^o^)/

## 未实现的方法

但是……等会儿，你可能注意到接口上还有很多的方法没有实现：

```csharp
SetReference(..)
AddReferenceToCollection(..)
RemoveReferenceToCollection(..)
SetConcurrencyValues(..)
```
因为目前为止我们还没有使用关系(Relationship)或者ETag功能，所以这些方法永远都不会被调用，你可以放心大胆的忽略这些方法。

不用担心，我会在后面的教程中逐步加入这些特性。

## 串联起来

最后一个步骤是修改我们的IServiceProvider的实现代码，当请求需要IDataServiceUpdateProvider接口时我们将构造一个实例并返回他。

## 总结

实现IDataServiceUpdateProvider的关键点是：延迟处理所有的请求。如果你使用代理原理也可以实现这些。

一旦你捕捉了所有的原子请求，在最后你只需要机械的重复再执行一边就可以了。
