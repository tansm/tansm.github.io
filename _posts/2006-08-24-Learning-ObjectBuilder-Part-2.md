---
title: "笨蛋学ObjectBuilder (二)"
date: 2006-08-24 11:51
---
Strategies.AddNew<
TypeMappingStrategy
>(
BuilderStage
.PreCreation);

类型映射策略，如果提供一个接口类型（通常是）和一个类类型（TypeMappingPolicy），当申请方要求创建一个接口类型时，实际创建的是那个类类型实例；

Strategies.AddNew<
SingletonStrategy
>(
BuilderStage
.PreCreation);

单实例策略，没有什么好说的；

Strategies.AddNew<
ConstructorReflectionStrategy
>(
BuilderStage
.PreCreation);

构造函数选择策略，决定调用哪个构造函数（还没有构造）

Strategies.AddNew<
PropertyReflectionStrategy
>(
BuilderStage
.PreCreation);

决定对哪些属性需要注射，还没有开始注射

Strategies.AddNew<
MethodReflectionStrategy
>(
BuilderStage
.PreCreation);

决定要调用哪些方法，还没有调用

Strategies.AddNew<
CreationStrategy
>(
BuilderStage
.Creation);

构造函数调用策略，实际调用构造函数以便创建实例

Strategies.AddNew<
PropertySetterStrategy
>(
BuilderStage
.Initialization);

属性注射策略，实际做属性的赋值操作；

Strategies.AddNew<
MethodExecutionStrategy
>(
BuilderStage
.Initialization);

方法调用策略，实际做方法的调用；

Strategies.AddNew<
BuilderAwareStrategy
>(
BuilderStage
.PostInitialization);

很特别的策略，检测创建后的对象是否符合IBuilderAware接口，如果是，也参与调用；

Policies.SetDefault<
ICreationPolicy
>(
new

DefaultCreationPolicy
());

加入默认的构造函数选择器，是根据参数的类型来选择
