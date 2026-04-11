---
title: "从LinQ看我们的ORM设计"
date: 2006-07-24 13:32
---
一直以来有些闭门造车，今天看了微软的一篇重要的ORM文章《[下一代数据访问：使概念级别成为现实](http://www.microsoft.com/china/MSDN/library/data/dataAccess/nxtgenda.mspx?mfr=true)》，详细的讲解了微软下一代数据访问的设想和实现。我对其中的关于概念模型的一段代码相当感兴趣，因为他和我设计的 Torridity（我们的 ORM 框架名）非常的相似。

## 关键字的描述的位置

在 MS 的关键字描述中，一个重要的改变是：关键字被描述到实体中，而不是字段中。

```xml
<!-- 典型的实体定义，具备标识 [关键字] 和一些成员 -->
<EntityType Name="Product" Key="ProductID">
  <Property Name="ProductID" Type="System.Int32" Nullable="false" />
</EntityType>
```

在很多 ORM 实现中，都是放在字段中，我认为那是错误的。现在看看我们的定义：

```xml
<dataEntityType name="Team" primaryKey="TeamId">
  <simpleProperties>
    <simpleProperty dbType="String" isUnique="False" index="False" name="TeamId" isBrowsable="False" />
    <simpleProperty dbType="String" isUnique="False" index="False" name="Code" />
    <simpleProperty dbType="String" isUnique="False" index="False" name="Name" />
  </simpleProperties>
</dataEntityType>
```

## 支持继承性

既然是做 ORM，继承性当然是不能缺少的，MS 使用下面的方式定义：

```xml
<!-- 派生产品，可以映射 TPH、TPC、TPT -->
<EntityType Name="DiscontinuedProduct" BaseType="Product">
  <Property Name="DiscReason" Type="System.String" Nullable="false" Size="max" />
</EntityType>
```

我们使用了相同的方式：

```xml
<dataEntityType name="ComparisonExpense" baseType="ComparisonBase">
  <simpleProperties>
    <simpleProperty dbType="Decimal" isUnique="False" index="False" name="EstimateExpense" />
    <simpleProperty dbType="Decimal" isUnique="False" index="False" name="ActualExpense" />
  </simpleProperties>
</dataEntityType>
```

## 支持复合类型

复合类型是对简单类型的扩展，实际上是支持对象化的属性。MS 的定义如下：

```xml
<!-- 复杂类型只定义结构而不定义标识。可以
在 0 个或更多实体定义中内嵌使用 -->
<ComplexType Name="CtAddress">
  <Property Name="Address" Type="System.String" Nullable="false" Size="max" />
  <Property Name="City" Type="System.String" Nullable="false" Size="max" />
  <Property Name="PostalCode" Type="System.String" Nullable="false" Size="max" />
  <Property Name="Region" Type="System.String" Nullable="false" Size="max" />
  <Property Name="Fax" Type="System.String" Nullable="false" Size="max" />
  <Property Name="Country" Type="System.String" Nullable="false" Size="max" />
  <Property Name="Phone" Type="System.String" Nullable="false" Size="max" />
</ComplexType>
```

```xml
<EntityType Name="Customer" Key="CustomerID">
  <!-- 地址是引用内嵌式复杂类型的成员 -->
  <Property Name="Address" Type="CNorthwind.CtAddress" Nullable="false" />
  <Property Name="CompanyName" Type="System.String" Nullable="false" Size="max" />
  <Property Name="ContactName" Type="System.String" Nullable="false" Size="max" />
  <Property Name="ContactTitle" Type="System.String" Nullable="false" Size="max" />
  <Property Name="CustomerID" Type="System.String" Nullable="false" Size="max" />
</EntityType>
```

复合类型在 Torridity 并没有被明确标明为 `ComplexType`，你可以使用任何实体作为结构复合进来。

```xml
<dataEntityType name="CallResult" primaryKey="CallResultId" isRoot="False">
  <simpleProperties>
    <simpleProperty dbType="String" isUnique="False" index="False" name="CallResultId" />
    <simpleProperty dbType="String" isUnique="False" index="False" name="ResultDescription" />
  </simpleProperties>
  <complexProperties>
    <complexProperty dataEntityType="PickListEntityCpx" name="Result" />
    <complexProperty dataEntityType="PickListEntityCpx" name="ObjectiveType" />
    <complexProperty dataEntityType="ProductCpx" name="MainProduct" />
  </complexProperties>
</dataEntityType>
```

## 支持集合属性

在 MS 的设计中，集合属性是这样描述的：

```xml
<!-- Product [如上定义] 和 OrderDetails [出于简明需要而未显示] 之间
关联的示例 -->
<Association Name="Order_DetailsProducts">
  <End Name="Product" Type="Product" Multiplicity="1" />
  <End Name="Order_Details" Type="OrderDetail" Multiplicity="*" />
</Association>
```

在 Torridity 中，一对一被称为引用属性，而一对多称为集合属性，使用下面的方式定义：

```xml
<dataEntityType name="Orders" baseType="QuotationBase" primaryKey="OrderId">
  <simpleProperties isExpansion="True">
    <simpleProperty dbType="String" isUnique="False" index="False" name="OrderId" />
    <referenceProperty referenceTo="Quotations" dbType="String" isUnique="False" index="False" name="QuatationId" />
    <simpleProperty dbType="Decimal" isUnique="False" index="False" name="TotalMargin" />
  </simpleProperties>
  <collectionProperties>
    <collectionProperty itemDataEntityType="RevenueAssignment" name="RevenueAssignments" />
  </collectionProperties>
  <complexProperties>
    <complexProperty dataEntityType="PickListEntityCpx" name="SourceId" />
  </complexProperties>
</dataEntityType>
```

注意引用属性的区别：MS 指向了一个类型，而我们指向了一个实体（TypeKey）。我认为 MS 的设计有些欠妥。

## 实体容器

在 MS 中支持实体容器，功能好像强一些：

```xml
<!-- 实体容器可定义
EntitySets（某一类型（可能）多形态实例的集合）和
AssociationSets（用于关联两个或更多实体实例的逻辑链接表）的
逻辑封装 -->
<EntityContainerType Name="CNorthwind">
  <Property Name="Products" Type="EntitySet(Product)" />
  <Property Name="Customers" Type="EntitySet(Customer)" />
  <Property Name="Order_Details" Type="EntitySet(OrderDetail)" />
  <Property Name="Orders" Type="EntitySet(Order)" />
  <Property Name="Order_DetailsOrders" Type="RelationshipSet(Order_DetailsOrders)">
    <End Name="Order" Extent="Orders" />
    <End Name="Order_Details" Extent="Order_Details" />
  </Property>
  <Property Name="Order_DetailsProducts" Type="RelationshipSet(Order_DetailsProducts)">
    <End Name="Product" Extent="Products" />
    <End Name="Order_Details" Extent="Order_Details" />
  </Property>
</EntityContainerType>
```

在 Torridity 中，集合类是自动创建的，没有专门的定义部分。

## 对接口的支持

我非常奇怪的问题，ORM 既然是对象化的，为什么没有包含一个非常重要的概念：接口。（也许作者忘记了吧）。

在 Torridity 提供了支持：

```xml
<dataEntityType name="Quotations" baseType="QuotationBase" primaryKey="QuotationId">
  <interfaces>
    <interface name="ICreateObject" />
    <interface name="IOwnerObject" />
    <interface name="INoteAccessoryObject" />
    <interface name="ICodeObject" />
    <interface name="IVersionObject" />
  </interfaces>
  <simpleProperties>
    <simpleProperty dbType="String" isUnique="False" index="False" name="QuotationId" />
    <simpleProperty dbType="DateTime" isUnique="False" index="False" name="StartDate" />
    <simpleProperty dbType="DateTime" isUnique="False" index="False" name="ExpireDate" />
  </simpleProperties>
</dataEntityType>
```

通过支持接口将大大简化实体的设计，而且是程序设计更加条例。我们知道真正的领域模型是不能通过继承性描述的，而支持通过特性（即接口）来描述。
