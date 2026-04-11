---
title: "笨蛋学ObjectBuilder"
date: 2006-08-23 17:20
---
为什么说是笨蛋学ObjectBuilder呢？并不是说这个文章是给笨蛋读的，而是一个笨蛋学习ObjectBuilder的笔记。

几个月前就开始研究ObjectBuilder了，但被他复杂的设计搞的没有头绪，现在总算懂了。

名词注解：

Strategy  直译是策略的意思，在这里是指对Object加工的一个操作；

Policy 直译是政策、方针，在这里指加工参数；

Locator 定位器，还没有透彻理解。

流程:

对象的建造过程非常类似一个产品流水线，ObjectBuilder是厂房容器，Strategy是流水线上的工人，负责不同的加工工艺，根据加工的阶段（BuilderStage）先后排放在流水线上，然后你可以事先准备一个图纸（Policy），以便工人们知道加工的参数，当然，你也可以在每个要 加工的对象旁放一个针对这个产品的加工图纸。

在Builder.Builder(IBuilderConfigurator<BuilderStage>)中准备默认的工人，存放到BuilderBase中Strategies属性中，Policies属性是存放长久有效的图纸(Policy)，在开始建造(BuildUp)时，你可以提供已经存在的对象和临时图纸。首先准备一个建造上下文，实际上就是ObjectBuilder信息的映射。另外还会合并长久图纸和临时图纸。加工从第一个员工开始，将上下文和其他信息传入，然后就不管了，流水线开始.....
