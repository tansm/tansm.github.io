---
title: "利用反射建立单一调用的WebService"
date: 2006-07-13 10:31
---
标题实在不好定义，是什么意思呢？我现在在做一个研究，就是原先的一个程序是用.NET Remoting进行远程通讯协议的，现在为了适应“广大客户关于WebService的强烈需求”，现在要修改为WebService方式。

稍微值得安慰的是：程序在设计的最初做了这种情况的假设，包含了一个通讯层，它在客户端包含一个连接对象，服务器端包含一个统一的服务调度程序，客户端总是返回一个服务的透明代理。开始人员在最上层调用看起来像这样的：

IEmployeeService service = Factory.GetService<IEmployeeService>();

- 要改成WebService方式，我必须在客户端也实现一个透明代理。

.NET默认建立的客户端是没有透明代理功能的。

关于透明代理的建立我参考了《[**A Taste of AOP from Solving Problems with OOP and Design Patterns (Part II)**](http://blog.joycode.com/jgtm2000/articles/13446.aspx)**》**一文，此高人2004年初就将AOP研究的透彻无比了。

- 利用反射统一调用方法

在服务器端，我先获取调用消息，WebService的定义如下：

    [WebMethod]

public object Invoke(string serviceType, string method, string[] argumentTypes, object[] arguments) {

利用反射机制，我找到这个服务的实例，并调用他。有些人会很担心安全问题，因为这样客户端就可以写恶意的调用了，在现有的程序中是没有这个问题，因为我限制了serviceType只能是我们指定的已经授权的服务。

- 关于CallContext

在.NET Remoting中，CallContext是一个非常有用的东西，他可以帮助我将上下文传输到服务端，但我做了WebService的试验，非常遗憾，客户端设置的上下文在服务器端无法获取到。现在还在研究。
