---
title: "关于获取服务的需求列表"
date: 2006-07-17 13:25
---
要获取服务，我们通常有哪几种需求呢？

- 获取指定接口的服务实现；

例如：

```csharp
ILogService service = Factory.GetService<ILogService>();
```

通常这种接口和实现是一一对应的。

- 获取某个TypeKey的特定接口的实现；

例如：

```csharp
ISaveService service = Factory.GetService<ISaveService>("Employee");
```

一个TypeKey和一个接口合起来才能定义一个实现。一个实现可能实现多个TypeKey，一个实现也会支持这个TypeKey的多个接口的实现。

- 获取特定接口的所有实现；

例如：

```csharp
IMoudleService[] services = Factory.GetServiceList<IMoudleSevice>();
```

这种方式的获取将遍历这个服务链。至于获取特定TypeKey的特定接口的所有实现的这种需求，还没有发现。
