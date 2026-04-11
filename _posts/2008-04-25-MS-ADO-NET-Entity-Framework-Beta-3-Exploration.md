---
title: "MS ADO.NET Entity Framework beta 3 探秘"
date: 2008-04-25 00:00:00 +0800
---

# MS ADO.NET Entity Framework beta 3 探秘

关于元数据结构

自动创建的代码包含一个派生自ObjectContext的对象，他有点像一个Workspace，主要包含连接、元数据和状态管理等；

ObjectContext包含一个MetadataWorkspace属性，记录了所有的元数据信息，有几种类型：

C  Space: 很全，包括数据类型（例如Byte）、函数(例如Max）、实体类型（例如Order）；

CS Space：null?没有吗

OC Space：纯数据类型，一般13个；

S  Space：包括数据类型和实体类型。

你可以像下面这样的代码获取元数据

EntityType c2 = c.MetadataWorkspace.GetItem<EntityType>("DcmsCore2_TestModel.Customer", System.Data.Metadata.Edm.DataSpace.CSpace);

枚举出一个元数据对象，怎么知道是描述什么的呢？BuildtInTypeKind属性可以告诉你，常见的有：

PrimitiveType    : 原生数据类型，例如描述string的

EntityType       ：实体类型，一般是ClrEntityType的实例，例如描述order

MetadataProperty : 属性，例如描述Order的属性OrderId；

Facet            ：特性，用来描述是否允许null，或缺省值之类的东西，有点像CLR里面的Attribute
