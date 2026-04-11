# U9在SQL Server上的性能优化经验（转述） — 之 行版本快照

> 原文：https://www.cnblogs.com/tansm/archive/2010/09/22/1833051.html

此文根据用友的文档《[基于SQL Server 2008构建SOA大型管理软件技术实践](http://wenku.baidu.com/view/b501ee00bed5b9f3f90f1c1e.html)》“翻译”而成，非原创。在baidu上看见此文，觉得写的很好，就将原先的PPT细化一下并除去废话。

  第二篇部分将的是行版本快照的隔离。

  这是PPT上的图片：

![Image](/images/2010-09-22-U9-SQL-Server-Performance-Row-Version-Snapshot-1.png)

 

  图片上已经非常明确的告诉我们，读取不会因为写而被阻塞，而是读取最后已经提交的数据。

  这在ERP应用中简直太有用了！！！

  以前，我们的客户在线用户数一多，就奇慢无比，并不是机器差，而是阻塞了。这个特性算是完美解决了。

  要了解如何启用此特性，请参考：《[SQL Server 中的快照隔离 (ADO.NET)](http://msdn.microsoft.com/zh-cn/library/tcbchxcb.aspx)》。其做法简而言之就是在数据库上执行：

```text
ALTER DATABASE MyDatabase
SET ALLOW_SNAPSHOT_ISOLATION ON

ALTER DATABASE MyDatabase
SET READ_COMMITTED_SNAPSHOT ON
```

以上两句话是设置快照功能启用，并设置默认的是读取快照事务级别。

 

## 我的观点：

虽然这是一个“老掉牙”的新特性了。但是我发现各大ERP厂商采用此技术的寥寥无几，并不是他们不知道此特性，而是… …没重视。这是我的理解哈，不要上火。希望这个小小的改动能够提高ERP的特性。

另外有人担心这是SQL Server 2005后的特性，其他的数据库怎么办？我想，在你的程序中写的稍微“复杂”些，多判断一下，你可能多花一天增加这个代码，但是你的客户却每时每刻享受此改进的时间缩短。
