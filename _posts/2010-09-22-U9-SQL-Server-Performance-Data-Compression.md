# U9在SQL Server上的性能优化经验（转述） — 之 数据压缩

> 原文：https://www.cnblogs.com/tansm/archive/2010/09/22/1833071.html

此文根据用友的文档《[基于SQL Server 2008构建SOA大型管理软件技术实践](http://wenku.baidu.com/view/b501ee00bed5b9f3f90f1c1e.html)》“翻译”而成，非原创。在baidu上看见此文，觉得写的很好，就将原先的PPT细化一下并添加废话。

   

  第三部分是讲数据压缩，U9采用的是页压缩。

![Image](/images/2010-09-22-U9-SQL-Server-Performance-Data-Compression-1.png)

  U9给出了他们的测试结果：

![Image](/images/2010-09-22-U9-SQL-Server-Performance-Data-Compression-2.png)

  通俗的说，通过压缩，SQL Server 可以提高近三倍的吞吐量（案例时间）。

  通过这个图形大家需要注意的是：**CPU并没有因为压缩变的高了，反而降低了一半**。

  要了解SQL Server的压缩功能如何启用，请参考：《[创建压缩表和索引](http://msdn.microsoft.com/zh-cn/library/cc280449.aspx)》。

## 我的观点

  没啥好说的，此功能对应用程序如此透明，干吗不用。

  当然，SQL Server 2008此功能如此好，为什么默认不开启呢？
