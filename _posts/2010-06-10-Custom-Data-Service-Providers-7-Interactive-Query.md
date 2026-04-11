# 自定义Data Service Providers — （7）交互式查询

> 原文：https://www.cnblogs.com/tansm/archive/2010/06/10/DSP7.html

完整教程目录请参见：《**[自定义Data Service Providers — 简介](http://www.cnblogs.com/tansm/archive/2010/06/10/1755250.html)**》

每当我编写一个插件并挂接到框架系统中时，我总是在想：框架系统是如何调用我的插件呢？

我已经习惯这种思维模式，它可以简化我理解其运行的原理。

事实上，在DSP开发过程中，我也是用这种方法理解DataService的调用过程。

下面我将使用**伪代码**来描述这些DSP接口是如何被调用的。

## 场景—基本查询

想想一下，一个客户端发起一个Get请求，例如： /Sample.svc/Products(1)。

首先发生的是Data Services的初始化事件。

```csharp
// 获取服务提供者
var dataservice = …;
IServiceProvider sp = dataservice as IServiceProvider;
if (sp == null)
{
// 以后再补充这里的代码 !!!
}
// 获取DSP各个接口的实现
IDataServiceMetadataProvider mdp =
sp.GetService(typeof(IDataServiceMetadataProvider));
IDataServiceQueryProvider qp =
sp.GetService(typeof(IDataServiceQueryProvider));
// 如有需要，赋值 CurrentDataSource 属性
if (qp.CurrentDataSource == null)
qp.CurrentDataSource = dataservice.CreateDataService();
//尝试获取名为 Products资源集
var resourceSet = null;
if (!mdp.TryResolveResourceSet(“Products”, out resourceSet))
throw new Exception(“404”);
// 获取资源集Products的查询对象（ queryable）
IQueryable queryRoot = qp.GetQueryRootForResourceSet(resourceSet);
// 使用URL上的选项（例如$filter/$select）在现有的查询对象上合并成新的查询表达式
queryRoot = Compose(options, queryRoot);
// 写入回复的“头”部分
WriteStartODataFeed();
// 枚举所有的查询结果
foreach (object resource in queryRoot)
{
// 这里需要获取资源对应的类型。
// 因为结果集中的类型可能是基础类型的派生类型，造成每个实例类型不一样。
ResourceType type = qp.GetResourceType(type);
WriteResource(resource,type);
}
WriteEndODataFeed();
```
希望通过这段伪代码能够帮助你理解DSP在Data Service框架中是如何工作的。

## 下一步

接下来的教程，我们将暂时离开这个查询模型，我们将开始实现服务操作和查询拦截器等内容。
