# 转载：《TypeScript 中文入门教程》 12、类型推导

> 文章转载自：https://github.com/zhongsp

## 介绍

这节介绍 TypeScript 里的类型推论。即，类型是在哪里如何被推断的。

## 基础

TypeScript 里，在有些没有明确指出类型的地方，类型推论会帮助提供类型。如下面的例子：

```typescript
var x = 3;  // x 的类型被推断为 number
```

这种推断发生在初始化变量和成员、设置默认参数值和决定函数返回值时。

## 最佳通用类型

当需要从几个表达式中推断类型时候，会使用这些表达式的类型来推断出一个最合适的通用类型：

```typescript
var x = [0, 1, null];  // 推断为 (number | null)[]
```

由于最终的通用类型取自候选类型，有些时候候选类型共享相同的通用类型，但是却没有一个类型能做为所有候选类型的类型：

```typescript
var zoo = [new Rhino(), new Elephant(), new Snake()];
// 想要 Animal[]，但没有 Animal 类型的对象，需要明确指定：
var zoo: Animal[] = [new Rhino(), new Elephant(), new Snake()];
```

如果没有找到最佳通用类型的话，类型推论的结果是空对象类型 `{}`。

## 上下文类型

TypeScript 类型推论也可能按照相反的方向进行，这被叫做"按上下文归类"：

```typescript
window.onmousedown = function(mouseEvent) {
    console.log(mouseEvent.buton);  // Error: 'buton' 不存在
};
```

TypeScript 类型检查器使用 `Window.onmousedown` 函数的类型来推断右边函数表达式的类型，因此能推断出 `mouseEvent` 参数的类型。

如果上下文类型表达式包含了明确的类型信息，上下文的类型被忽略：

```typescript
window.onmousedown = function(mouseEvent: any) {
    console.log(mouseEvent.buton);  // 不报错，因为明确指定了 any
};
```

上下文归类会在很多情况下使用到，通常包含函数的参数、赋值表达式的右边、类型断言、对象成员和数组字面量和返回值语句。
