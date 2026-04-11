---
title: "对可二次开发的软件结构的探究"
date: 2006-05-10 16:59
---

# 对可二次开发的软件结构的探究

目前公司写的程序已经告一段落了，这个程序从一开始就定位为可二次开发的系统，作为商业应用程序，二次开发的主要需求包括：

－ 所有二次开发理论上不修改任何现有代码和发布方式，以及不用公开源代码；
－ 添加新的业务实体，以及对应的商业服务和界面；
－ 现有实体中添加新的字段，增加新的业务校验、在已有流程中增加新的动作，以及在已有界面中增加新的表现；
－ 重构已有的实现；
－ 所有的组件都可以组合和拆分，保证仍然能够工作正常。
－ 公司内部的开发和二次开发具有相同的模型和体验。

这些需求将直接影响产品设计的方方面面，现在这个平台的第一个作品已经开发到beta，现在开始准备二次开发的实践。因为涉及到公司的机密，所以不能讲实现原理，我这里将讲解二次开发的体验部分。

首先二次开发商建立独立的DLL，引入实体和服务接口库；
然后为新的实体添加字段，例如：

```
[DataEntity("AccountId",TypeKey="Account")]
[Serializable]
public class AccountEx : Account {

private string _newField;
/**////
/// 返回/设置新的扩展字段
///
public string NewField {
get { return _newField; }
set {
if (_newField != value) {
_newField = value;
OnPropertyChanged("NewField");
}
}
}
}
```

在这里例子中，新的实体继承了已有实体，并添加了新的字段：NewField。但注意，实体仍然标记为Account以便保持兼容。

下面再介绍如何添加新的商业规则。
建立一个插件，标明对应到Account服务，并在初始化时拦截原有服务的SaveBefore事件。

```
[ServiceAddin(Account.TYPE_KEY)]
public class AccountServiceAddin : BusinessServiceAddin> {

protected override void InitializeComponent() {
base.InitializeComponent();
Owner.SaveBefore += new System.EventHandler.SaveBeforeArgs>(Owner_SaveBefore);
}

void Owner_SaveBefore(object sender, BusinessServiceBase.SaveBeforeArgs e) {
AccountEx newObject = (AccountEx)e.DataEntity;
newObject.NewField = DateTime.Now.ToString();
}
}
```

在这里例子中简单的为NewFiled赋了新的值。

关于界面的可扩展性将在后面的文章中介绍，总结以上的开发体验，大多数功能都已经实现，但是最后一条关于公司内部和二次开发商具有相同的开发体验方面做的很不好，主要表现为：
－ 新加的字段继承了已有实体Account，更好的方法是建立独立的实体，与已有的实体形成“桶装”方式；
－ 新扩展的商业规则使用了插件的方式，但公司内部的开发却使用了继承并重载的模式；

相信很多的架构师在此方面都有很多的经验，希望可以给予指导。
路漫漫......
