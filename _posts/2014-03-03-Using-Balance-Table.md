# 余额表的处理方法

> 原文：https://www.cnblogs.com/tansm/p/UsingBanlanceTable.html

# 原由

在开发ERP应用中，我们经常需要知道某个实体的当前数量，例如知道商品当前的库存，或者科目的金额，或者某个客户剩余的信用额度，所以这种需求是比较普遍的。

通常会设计两张表，一张是流水账表，有的称明细表，或者日志表，用于记录所有发生的事务记录。一张是余额表，用于记录各个实体当前最新的余额。

 

**流水号**

**NO.**

**商品编号**

**ProductId**

**增加**

**Add**

**减少**

**Reduce**

**1**

P1

3

 

**2**

P1

 

1

**3**

P2

5

 

 

**商品编号**

**ProductId**

**余额**

**Banlance**

**P1**

2

**P2**

5

 

如果有一张单据分别出货P1和P2各一件，而另外一个单据也出货P2和P1各一件，只是顺序不同而已。所以可能出现下面的sql执行顺序。

 

**顺序**

**Order**

**用户1**

**User 1**

**用户2**

**User 2**

**状态**

**Status**

**1**

begin tran

begin tran

OK

**2**

```text
INSERT INTO [Chronological]

([NO],   [ProductId],[Reduce])

VALUES   (4, ‘P1’, 1);
```

```text
INSERT INTO [Chronological]

([NO],   [ProductId],[Reduce])

VALUES (5, ‘P2’, 1);
```

OK

**3**

```text
Update   [Banlance]

  Set Banlance = Banlance- 1

  Where ProductId = ‘P1’
```

 

OK

**4**

 

```text
Update [Banlance]

  Set Banlance =   Banlance- 1

  Where ProductId =   ‘P2’
```

OK

**5**

```text
Update   [Banlance]

  Set Banlance = Banlance- 1

  Where ProductId = ‘P2’
```

 

Wait…

**6**

 

```text
Update [Banlance]

  Set Banlance =   Banlance- 1

  Where ProductId =   ‘P1’
```

Wait

 

可以看出，当两个用户需要的资源（余额表）很容易产生死锁，这是本文要关键解决的问题。

另外，实际情况还要更复杂，由于不能保证余额表一定存在对应的产品记录，当Update Banlance表未影响任何行时，需要执行一个Insert 指令，你很容易想到的，另外一个用户也正好执行了此产品的Update，也发现未影响任何行兵执行一个Insert指令，而后执行的insert将出现“数据重复”的异常。

# 解决方案一：顺序的更新数据

第一种解决方案是在执行Update指令时，对要更新的数据进行排序。对于此案例，我们可以对ProductId进行排序，重新执行上面的SQL.

**顺序**

**Order**

**用户1**

**User 1**

**用户2**

**User 2**

**状态**

**Status**

**1**

```text
begin tran
```

```text
begin tran
```

OK

**2**

```text
INSERT INTO [Chronological]

([NO],   [ProductId],[Reduce])

VALUES   (4, ‘P1’, 1);
```

```text
INSERT INTO [Chronological]

([NO],   [ProductId],[Reduce])

VALUES (5, ‘P1’, 1);
```

OK

**3**

```text
Update   [Banlance]

  Set Banlance = Banlance- 1

  Where ProductId = ‘P1’
```

 

OK

**4**

 

```text
Update [Banlance]

  Set Banlance =   Banlance- 1

  Where ProductId =   ‘P1’
```

Wait…

**5**

```text
Update   [Banlance]

  Set Banlance = Banlance- 1

  Where ProductId = ‘P2’
```

 

OK

**6**

```text
commit
```

 

OK

**7**

 

-- 指令 4   开始执行

OK

**8**

 

```text
Update [Banlance]

  Set Banlance =   Banlance- 1

  Where ProductId =   ‘P2’
```

OK

**9**

 

```text
commit
```

OK

通过排序，我们很好的避免了死锁问题。

实际情况是可能没有对应的商品行，需要提前插入数据，让我们看看没有P1的情况。（为简化示例，省去流水账的SQL，改出库为入库，仅入库P1）。

**顺序**

**Order**

**用户1**

**User 1**

**用户2**

**User 2**

**状态**

**Status**

**1**

```text
begin tran
```

```text
begin tran
```

OK

** **

…

…

 

**2**

```text
Update   [Banlance]

  Set Banlance = Banlance + 1

  Where ProductId =   ‘P1’
```

-- 没有影响行

OK

**3**

 

```text
Update [Banlance]

  Set Banlance =   Banlance + 1

  Where ProductId =   ‘P1’

-- 没有影响行
```

OK

**4**

```text
Insert Into [Banlance]

         (ProductId,Banlance)

         Values(‘P1’,1);
```

 

OK

**5**

 

```text
Insert Into [Banlance]

         (ProductId,Banlance)

         Values(‘P1’,1);
```

Wait

**6**

```text
commit
```

 

OK

**7**

 

-- 指令 5 开始执行

-- 重复的键

Error

**8**

 

```text
-- 已存在记录，改执行Update

Update [Banlance]

  Set Banlance =   Banlance + 1

  Where ProductId =   ‘P1’
```

OK

 

**9**

 

```text
commit
```

OK

由于第二个insert语句插入相同主键的数据，所以出现等待，当用户1提交后，可以发现用户2执行的insert会失败，改执行update语句。这样也能解决问题，只是非常蹩脚。

如果你使用SQL Server 2008及其以后的版本，可以使用新的语法解决这个问题。

**顺序**

**Order**

**用户1**

**User 1**

**用户2**

**User 2**

**状态**

**Status**

**1**

begin tran

begin tran

OK

** **

…

…

 

**2**

```text
MERGE [Banlance]   AS target

    USING (SELECT ‘P1’) AS source (ProductId)

    ON (target.ProductId = source.ProductId)

    WHEN MATCHED THEN

        UPDATE SET Banlance = Banlance + 1

         WHEN NOT MATCHED THEN

               INSERT (ProductId, Banlance)

        VALUES (‘P1’,1);
```

 

OK

**3**

 

```text
MERGE [Banlance] AS target

    USING (SELECT   ‘P1’) AS source (ProductId)

    ON   (target.ProductId = source.ProductId)

    WHEN MATCHED   THEN

        UPDATE SET Banlance   = Banlance + 1

         WHEN NOT   MATCHED THEN

             INSERT (ProductId, Banlance)

             VALUES (‘P1’,1);
```

Wait

**4**

commit

 

OK

**5**

 

commit

OK

第一个用户会插入数据，第二个用户会执行更新操作。

# 解决方案二：废弃余额表

第二个方案直接废除余额表，这样就从根本上避免了资源的死锁问题。当然，没有了余额表，要获知当前余额，就需要稍微改动现在的流水表。

**流水号**

**NO.**

**期间**

**Period**

**来源**

**Source**

**商品编号**

**ProductId**

**增加**

**Add**

**减少**

**Reduce**

**1**

2

 

P1

10

 

**2**

2

 

P2

5

 

**3**

2

S1

P1

3

 

**4**

2

S2

P1

 

1

**5**

2

S1

P2

5

 

创建表的SQL如下：

```text
CREATE TABLE [dbo].[Chronological] (

    [NO]        INT           NOT NULL,

    [Period]    INT           NOT NULL,

    [Source]    NVARCHAR (20) NOT NULL,

    [ProductId] INT           NOT NULL,

    [Add]       DECIMAL (18)  NOT NULL,

    [Reduce]    DECIMAL (18)  NOT NULL,

    PRIMARY KEY CLUSTERED ([NO] ASC)

);
```

 

通过将相同商品编号的数据分组并sum，即可获取其剩余的库存，例如获取P1的库存。

```text
select Top 1 ProductId,sum([Add]) - sum([Reduce]) from Chronological where ProductId = ’P1’ and Period = 2
       group by ProductId ;
```

 

这里我们使用了区间(Period)来减少数据汇总对应的计算量。来源（Source）如果是空，那么表示此行是期初数据，他在结转期间时创建。

如果你觉得这样做还不够，也可以设计一张流水历史表，将所有非当前区间的数据放在流水历史表，这样当前表（Chronological）的数据量就比较小了，也不要区间(Period)字段了。

另外，如果你加入【发生日期】字段，将达到另外一个好处，可以查询任意时间点的库存了，而不必像之前余额表那样仅能查询当前库存。

最后，如果你希望出库时不允许负库存怎么办？由于Insert执行不能进行检查，再下一条SQL会不会造成查询的库存错误的统计到别人尚未提交的库存记录呢？其实不必担心，只要在查询库存时仅包括比自己记录小的记录即可。

**顺序**

**Order**

**用户1**

**User 1**

**用户2**

**User 2**

**状态**

**Status**

**1**

begin tran

begin tran

OK

**2**

```text
insert into [Chronological]

([NO],[Period],[Source],[ProductId],[Add],[Reduce])

  --OUTPUT   INSERTED.[NO]

  values(6,2,'S3',‘P2’,0,9);
```

 

OK

**3**

 

```text
insert into   [Chronological]

([NO],[Period],[Source],[ProductId],[Add],[Reduce])

  --OUTPUT INSERTED.[NO]

  values(7,2,'S4',‘P2’,0,5);
```

OK

**4**

```text
select TOP 1 ProductId,sum([Add]) - sum([Reduce]) from   Chronological

with(NOLOCK)

 where ProductId =   ‘P1’ and Period = 2 and [NO] <= 6

         group by   ProductId ;
```

 

OK

**5**

 

```text
select TOP 1 ProductId,sum([Add])   - sum([Reduce]) from Chronological

with(NOLOCK)

 where ProductId = ‘P1’ and Period = 2 and   [NO] <= 7

         group by ProductId ;
```

wait

或OK

**6**

commit

 

OK

**7**

 

rollback

OK

注意的细节是我在select时加入了 with (NOLOCK)，数据库默认是没有启用“已提交快照读”，所以在执行第5句话时会出现wait，从而在执行6后，访问的是最新的结果。但如果数据库启用了“已提交快照读”，那么第5句不会wait，且返回5这个不正确的结果。所以我在sql中加入了with(NOLOCK)保证无论何种情况下都检查正确。
