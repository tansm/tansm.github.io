# 关于ADO.NET Entity Framework的信息

> 原文：https://www.cnblogs.com/tansm/archive/2008/01/29/1057202.html

今天逛了一下ADO.NET Entity Framework团队的Blog，我对这个项目比较感兴趣，因为我也是写ORM的。:) 当然了，那档次不是一个级别的。

这个项目我都不知道放了多少次鸽子了，我还记得ObjectSpace这个东西，到现在出到beta 3,连LinQ都发表了，他总不能再放鸽子吧。（Vista中WinFS我猜想也是这个东西他们的放鸽子，所以WinFS也不得不放鸽子了。）

好，言归正传，有那些东西呢？

首先是[Beta 3发布的文章](http://blogs.msdn.com/adonet/archive/2007/12/05/ado-net-entity-framework-beta-3-rleased.aspx)（不能算新闻了），有

- 性能的改进；

- 为商业逻辑支持提供必要的事件；

- 查询的改进，编译的LinQ支持，提供ToReaceString()方法方便调试；

- 更好的对其他数据库的支持；

关于设计器方面：

[Entity Designer CTP2](http://blogs.msdn.com/adonet/archive/2007/12/06/entity-designer-ctp2.aspx)

[Entity Data Model Designer Video - CTP 2](http://blogs.msdn.com/adonet/archive/2007/12/13/entity-data-model-designer-video-ctp-2.aspx) 在ADO.NET Entity Framework中如何使用存储过程；

[Customizing Code Generation in the ADO.NET Entity Designer](http://blogs.msdn.com/adonet/archive/2008/01/24/customizing-code-generation-in-the-ado-net-entity-designer.aspx)

当然，还有很多的教程之类的

[SampleEdmxCodeGenerator sources](http://blogs.msdn.com/adonet/archive/2008/01/24/sampleedmxcodegenerator-sources.aspx)

[Programming LINQ and the ADO.NET Entity Framework Webcast](http://blogs.msdn.com/adonet/archive/2008/01/28/programming-linq-and-the-ado-net-entity-framework-webcast.aspx)

[How does the ADO.NET Entity Designer generate code?](http://blogs.msdn.com/adonet/archive/2008/01/24/how-does-the-ado-net-entity-designer-generate-code.aspx)

[How to Extract CSDL from EDMX](http://blogs.msdn.com/adonet/archive/2008/01/24/how-to-extract-csdl-from-edmx.aspx)

[Annotations in CSDL](http://blogs.msdn.com/adonet/archive/2008/01/11/annotations-in-csdl.aspx)

当然，我最有兴趣的是这个：

[ADO.NET performance improvements with the .NET Framework 2.0 SP1](http://blogs.msdn.com/adonet/archive/2008/01/28/ado-net-performance-improvements-with-the-net-framework-2-0-sp1.aspx)

他上面说，.NET FW 2.0 Sp1将SqlReader读取性能从14855增加到18100，不是一般的性能提高啊。
