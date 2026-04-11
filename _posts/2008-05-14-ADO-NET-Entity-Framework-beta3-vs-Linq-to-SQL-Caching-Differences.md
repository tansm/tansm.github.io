---
title: "ADO.NET Entity Framework beta 3 和Linq to SQL 在缓存处理上的不同"
date: 2008-05-14 00:00:00 +0800
---

# ADO.NET Entity Framework beta 3 和Linq to SQL 在缓存处理上的不同

Linq to SQL内置缓存功能，简单的说，当你查询了一次某个键的数据后，再次查询时linq to SQL的引擎不再向数据库发送SQL，例如：

```text
//下面是使用LinQ to SQL 的例子，context2是派生自System.Data.Linq.DataContext的实例

            ErpLinQContextDataContext context2 = new ErpLinQContextDataContext();

            //SQLServer事件探查器拦截到SQL语句的执行

            //exec sp_executesql N'SELECT TOP 1 [t0].[emp_id], [t0].[fname], [t0].[minit], [t0].[lname],

            //                         [t0].[job_id], [t0].[job_lvl], [t0].[pub_id], [t0].[hire_date]

            //FROM [dbo].[employee] AS [t0]

            //WHERE [t0].[emp_id] = @p0', N'@p0 varchar(9)', @p0 = 'PMA42628M'

            employee p3 = context2.employees.First*(p => p.emp_id == "PMA42628M");

            //当我再次执行相同的查询时，LinQ to SQL 不再向SQL Server发送查询了。

            employee p4 = context2.employees.First*(p => p.emp_id == "PMA42628M");
```

而且，这两个实例是同一个实例：

```text
//返回的对象是同一个实例

            bool b2 = object.ReferenceEquals(p3, p4); //=true;

            p3.lname = "New Last Name";

            bool b4 = (p4.lname == "New Last Name");  //=true;
```

当然，如果你使用不同的Context实例查询时，缓存功能将实效。

好，让我们再看看ADO.NET Entity Framework beta 3：

```text
//pubsEntites是 ADO.NET Entity Framework 的System.Data.Objects.ObjectContext派生对象

            pubsEntities context = new pubsEntities();

            //下面语句执行时，SQLServer事件探查器拦截到SQL的执行

            //SELECT TOP 1 [Extent1].[emp_id] AS [emp_id], [Extent1].[fname] AS [fname], [Extent1].[lname] AS [lname],

            //             [Extent1].[hire_date] AS [hire_date], [Extent1].[job_id] AS [job_id], [Extent1].[pub_id] AS [pub_id]

            //FROM [dbo].[employee] AS [Extent1]

            //WHERE N'PMA42628M' = [Extent1].[emp_id]

            Employee p1 = context.EmployeeSet.First*(p => p.EmployeeId == "PMA42628M");

            //SQLServer事件探查器 发现SQL再次被执行

            Employee p2 = context.EmployeeSet.First*(p => p.EmployeeId == "PMA42628M");

            //测试发现，虽然ADO.NET Entity Framework执行了两次SQL，但是他们却返回了完全相同的实例

            bool b1 = object.ReferenceEquals(p1, p2); // = true;
```

测试的结果是，ADO.NET Entity Framework（以下简称AEF）没有使用缓存，而是再次执行SQL，但是你要注意：两次查询的实例竟然是同一个。

从Context功能上看，他肯定持有上次查询的结果，他没有使用缓存，我只能认为可能AEF被设计成三层应用，那么他很担心其他的进程将数据改了，所以不使用缓存，当发现数据并没有改后，还是使用原先的实例，这个想法对吗？

我们再看看另外一个代码：

```text
//如果使用不同的上下文更新的数据，

            pubsEntities context4 = new pubsEntities();

            Employee p10 = context4.EmployeeSet.First*(p => p.EmployeeId == "PMA42628M");

            p10.LastName = "Context4 changed data";

            context4.SaveChanges();

            //旧的context再次查询时。

            Employee p11 = context.EmployeeSet.First*(p => p.EmployeeId == "PMA42628M");

            b1 = object.ReferenceEquals(p1, p11); //= true   why??

            b1 = (p11.LastName == "Context4 changed data"); //= false  p11.LastName = "New Last Name"
```

难以置信，AEF重新执行了SQL，但是置新的更改而不闻，仍然返回旧的数据。这个算是Bug吗？

我不知道哪位达人能够解释这个问题？当然，这个问题我也询问了MS，他们的技术人员还未做出满意的答复。
