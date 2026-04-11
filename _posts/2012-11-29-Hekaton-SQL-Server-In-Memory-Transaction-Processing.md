# 转发：Hekaton:SQL Server集成的内存事务处理

> 原文：https://www.cnblogs.com/tansm/archive/2012/11/29/2794859.html

原文地址：[http://www.infoq.com/cn/news/2012/11/Hekaton](http://www.infoq.com/cn/news/2012/11/Hekaton)，觉得很好的新闻，专此大家分享，红字是我觉得很激动的设计。我还找到[微软介绍此技术的文章。](http://blogs.technet.com/b/dataplatforminsider/archive/2012/11/08/breakthrough-performance-with-in-memory-technologies.aspx)

作者 **[Abel Avram](http://www.infoq.com/cn/author/Abel-Avram) **译者 **[孙镜涛](http://www.infoq.com/cn/author/%E5%AD%99%E9%95%9C%E6%B6%9B) **发布于 2012年11月27日

 

在[2012年度的SQL Server专业协会（PASS）峰会](http://www.microsoft.com/sqlserver/en/us/default.aspx)（SQL Server专业人士的技术会议）上，微软发布了Hekaton，一个针对事务处理（TP）的基于行的内存数据管理系统。除了宣传的为遗留应用程序提升10倍的TP速度、为新优化的应用提升50倍的速度之外，微软还透露，正在将Hekaton 完全集成进SQL Server。

现有应用程序也可从这一数据库新技术中受益，首先通过工具的帮助确定最常用的表，然后配置服务器将这些表存储到系统主内存里，经过内存优化的数据结构可获取更快的访问时间，而剩余的数据可以存储在传统的经过存储优化的数据结构中，并在需要时调入或调出磁盘。

此前，SQL Server一直对查询语句和存储过程进行编译，将它们转变成由查询处理器解析的数据结构，但是Hekaton则将T-SQL存储过程直接编译成本地代码从而加快执行速度。

Hekaton使用一个新的由微软和威斯康星大学的研究者共同开发的并发控制机制（[PDF](http://vldb.org/pvldb/vol5/p298_per-akelarson_vldb2012.pdf)），该机制通过无锁的数据结构在多核心之间获得更好的伸缩性，避免锁的同时保留了ACID事务完整性。

日前，Hekaton正在由微软选定的合作伙伴进行测试，它将会包含在SQL Server的下一个主版本中。在Hekaton发布之后，它将与[SAP Hana](http://en.wikipedia.org/wiki/SAP_HANA) 以及 [Oracle Exadata X3](http://www.oracle.com/us/products/database/exadata/overview/index.html)进行竞争，前者是2010年发布的一个独立的用于实时分析的设备和数据库，后者是运行着Oracle数据库11g的一个设备，它将热点数据保存到固态存储中。

微软还宣布新版[SQL Server 2012 并行数据仓库 （PDW）](https://www.microsoft.com/sqlserver/en/us/solutions-technologies/data-warehousing/pdw.aspx)将于2013年上半年发布，这是一个SQL Server 设备。

**英文原文地址**：[http://www.infoq.com/news/2012/11/Hekaton](http://www.infoq.com/news/2012/11/Hekaton)
