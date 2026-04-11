---
title: "真正的领域模型是不能通过继承性描述的"
date: 2006-07-25 15:33
---
[双鱼座](http://barton131420.cnblogs.com/)在我的一篇《[从LinQ看我们的ORM设计](http://tansm.cnblogs.com/archive/2006/07/24/458263.html)》一文中的评论中，对“真正的领域模型是不能通过继承性描述的”提出了质疑。

在说明我的理由之前，我需求先向他道歉，因为我武断的说他“抱着课本是天书”。

OK，回到话题，假设我们正在开发一个进销存系统，我们总结出所有的文档都需要修改记录(RichDocument)，而所有的树都需要ParentId（TreeDocument），而大多数的文档需要Code字段（CodeDocument）。

![](/images/Domain-Model-Cannot-Be-Described-by-Inheritance-1.PNG)

现在有个实体A需要修改记录功能和编码功能，因为.NET不支持多根对象，所以必须派生出一个CodeRichDocument。复制CodeDocument的字段，现在又有一个实体B需要修改记录功能和树功能，于是我们又派生了一个TreeRichDocument。不幸的是，又有一个实体C三个功能都要，如何派生？从CodeRichDocument继承，然后再加一个ParentId字段？这样的话，新的派生类就 Not Is TreeDocument了。

![](/images/Domain-Model-Cannot-Be-Described-by-Inheritance-2.PNG)

这还是才三个“特性”，实际的业务要比这个复杂的多，“不讲道理”的多。

如何解决？我们使用接口，将每个需要的功能称为“特性”，使用接口描述，例如：

![](/images/Domain-Model-Cannot-Be-Described-by-Inheritance-3.PNG)

这样C Is IRichDocument And Is ICodeDocument And ITreeDocument.
