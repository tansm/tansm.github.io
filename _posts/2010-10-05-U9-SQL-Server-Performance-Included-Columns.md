# U9在SQL Server上的性能优化经验（转述） — 之 包含列索引

> 原文：https://www.cnblogs.com/tansm/archive/2010/10/05/1843241.html

此文根据用友的文档《[基于SQL Server 2008构建SOA大型管理软件技术实践](http://wenku.baidu.com/view/b501ee00bed5b9f3f90f1c1e.html)》“翻译”而成，非原创。在baidu上看见此文，觉得写的很好，就将原先的PPT细化一下并添加废话。

  第四部分，讲的是“包含列索引”的使用。

  关于此功能的MSDN说明请参考：[http://msdn.microsoft.com/zh-cn/library/ms190806(SQL.90).aspx](http://msdn.microsoft.com/zh-cn/library/ms190806(SQL.90).aspx)

  另外，在网络上发现了这篇很好的文章：[SQL Server 索引中include的魅力（具有包含性列的索引）](http://www.cnblogs.com/gaizai/archive/2010/01/11/1644358.html)

  [

![Image](/images/2010-10-05-U9-SQL-Server-Performance-Included-Columns-1.png)

](http://images.cnblogs.com/cnblogs_com/tansm/Windows-Live-Writer/daa94461fc5a_8FE5/image_2.png)

## 我的观点：

  以下是我的“瞎猜”。需要在我的机器安装SQLServer才能证明以下。

  普通索引仅仅记录对应到行的位置（以及索引自身的数据），想象一下我们经常使用的一个SQL语句：

  Select Name From Employee Where Code = @Code

  如果索引IDX_Code没有使用包含列，那么引擎会通过此索引寻找到数据，如果此索引包含列Name，那么就会直接通过索引找到Name数据。

  我可以想象以下场景的SQL语句会用到包含列的索引：

  1、通过编号获取查询获取名称标题；

  2、通过编码获取名称；

  3、在诸如树控件上显示名称；
