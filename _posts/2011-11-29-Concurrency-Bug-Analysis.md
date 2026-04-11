# 一个并发环境下的BUG分析

> 原文：https://www.cnblogs.com/tansm/archive/2011/11/29/2268193.html

今天接手一个并发压力下的一个BUG，程序报：“字典中已经添加了重复的键”，出错的代码如下：

```text
lock (connectionList){    // Next we'll see if there is already a connection. If not, we'll create a new connection and add it    // to the transaction's list of connections.    // This collection should only be modified by the thread where the transaction scope was created    // while the transaction scope is active.    // However there's no documentation to confirm this, so we err on the safe side and lock.    if (!connectionList.TryGetValue(db.ConnectionString, out connection))    {        // we're betting the cost of acquiring a new finer-grained lock is less than         // that of opening a new connection, and besides this allows threads to work in parallel        var dbConnection = db.GetNewOpenConnection();        connection = new DatabaseConnectionWrapper(dbConnection);        connectionList.Add(db.ConnectionString, connection);    }    connection.AddRef();}
```

这段代码是企业库的一段代码（参见：[http://entlib.codeplex.com/SourceControl/changeset/view/90009#526470](http://entlib.codeplex.com/SourceControl/changeset/view/90009#526470)）

那个Add方法出错的，我得承认，我花了很长时间想这段代码在哪种情况下会造成锁定失败呢？怎么可能有两个线程进来。你看出来了吗？

好吧，答案是，问题并不是在lock上，而是在两次调用db.ConnectionString上。

在并行环境下，外界错误的使用了静态变量共享了IDatabase实例（这个db对象），这是主因。但是在这段代码中，判断字典是否存在某个key时使用了db.ConnectionString获取，但很不幸，在并行环境下，这个静态共享的db对象被其他线程替换成了另外一个连接字符串，于是在Add时，使用了另外一个字符串作为key,又不幸的是，字典中已经存在了此键。

修改的方法当然是先取消外部静态共享IDatabase对象。但这个地方也不妥，严密的代码应该是：

```text
string key = db.ConnectionString;lock (connectionList){    // 使用同一个Key值，而不是两次访问属性，可能不一致。    if (!connectionList.TryGetValue(key, out connection))    {        var dbConnection = db.GetNewOpenConnection();        connection = new DatabaseConnectionWrapper(dbConnection);        connectionList.Add(key, connection);    }    connection.AddRef();}
```

总结：

1、慎用静态变量，他经常是内存泄露、并发冲突的元凶；

2、注意属性可能在并发下中途被修改，可以使用字段备份一份。
