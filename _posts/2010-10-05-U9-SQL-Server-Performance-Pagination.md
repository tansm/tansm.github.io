# U9在SQL Server上的性能优化经验（转述） — 之 分页

> 原文：https://www.cnblogs.com/tansm/archive/2010/10/05/1844234.html

此文根据用友的文档《[基于SQL Server 2008构建SOA大型管理软件技术实践](http://wenku.baidu.com/view/b501ee00bed5b9f3f90f1c1e.html)》“翻译”而成，非原创。在baidu上看见此文，觉得写的很好，就将原先的PPT细化一下并添加废话。

  第五部分讲的是分页，这是原先的分页语句：

![Image](/images/2010-10-05-U9-SQL-Server-Performance-Pagination-1.png)

  从最后的？！可以看出，当时U9在此SQL语句上费了很多的CPU，（才几十万条记录就搞不定了？）。

  好了，U9优化后的语句粉墨登场了：

![Image](/images/2010-10-05-U9-SQL-Server-Performance-Pagination-2.png)

  PPT上一个“ ！” 充分说明了优化的效果让U9提高很多。

  显然，之后的SQL语句将数据范围减少后再关联，所以提高了效率，可为什么会出现当时的错误，我想可能是MSDN错误的引导吧，因为包括我自己在内之前也是这么写的。

## 我的观点

  进一步的，我在想，权限的Where条件到底是放在With as的表达式中还是放在外面的SQL语句中呢？答案应该是表达式里面，否则输出的一页就不够数了。推而广之，所有的条件应该都在里面了。

  另外一个需要注意的地方，如果排序条件要使用到别的表，还是要关联其他的表，对于ERP这样的自动化引擎，应该还是一个挑战的。
