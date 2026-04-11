# 《TypeScript 中文入门教程》 1、基础数据类型

> 转载自：https://github.com/MyErpSoft/TypeScript-Handbook/blob/master/pages/zh-CHS/Basic%20Types.md

## 概述

为了让程序更易用，我们兼容几种最基本的数据类型：numbers(数字)，strings(字符串)，structures(结构)，boolean(布尔值)等等。在 TypeScript 中，我们支持和 Javascript 几乎一样多的类型，并且新增了实用的枚举类型。

## Boolean 布尔值

最基础的数据类型就是简单的 true/false，在 Javascript 和 TypeScript 中被称作 "boolean(布尔值)"。

```typescript
var isDone: boolean = false;
```

## Number 数字

和 Javascript 一样，在 TypeScript 中所有的 number 都是浮点值。TypeScript 除了支持十六进制和十进制外，还支持二进制和八进制。

```typescript
var decLiteral: number = 6;
var hexLiteral: number = 0x9837abdef;
var binaryLiteral: number = 0b0010;
var octalLiteral: number = 0o74563;
```

## String 字符串

和 JavaScript 一样，TypeScript 也使用双引号或单引号来围绕字符串数据。

```typescript
var name: string = "bob";
name = 'smith';
```

你也可以使用模板字符串，支持多行文本和内嵌表达式：

```typescript
var name: string = `Gene`;
var age: number = 37;
var sentence: string = `Hello, my name is ${ name }.
I'll be ${ age + 1 } years old next month.`
```

## Array 数组

TypeScript 允许你使用数组，有两种写法：

```typescript
var list: number[] = [1, 2, 3];
var list: Array<number> = [1, 2, 3];
```

## Tuple 元组

元组类型允许表达固定数量的已知类型集合：

```typescript
var x: [string, number];
x = ['hello', 10]; // 正确
x = [10, 'hello']; // 错误
```

## Enum 枚举

TypeScript 拓展了 JavaScript 原生的标准数据类型集，增加了枚举类型：

```typescript
enum Color {Red, Green, Blue};
var c: Color = Color.Green;

// 从1开始计数
enum Color {Red = 1, Green, Blue};

// 通过数值查找名称
enum Color {Red = 1, Green, Blue};
var colorName: string = Color[2]; // "Green"
```

## Any

当我们需要描述类型不明确的变量时，可以使用 `any` 类型：

```typescript
var notSure: any = 4;
notSure = "maybe a string instead";
notSure = false;
```

## Void

`void` 就像 `any` 的相反面，没有返回值的函数可以认为是 `void` 类型：

```typescript
function warnUser(): void {
    alert("This is my warning message");
}
```

---

感谢翻译：
- oyyd https://github.com/oyyd/typescript-handbook-zh
- ntesmail https://github.com/ntesmail/Typescript-Handbook
- 编写人生 https://github.com/MyErpSoft/TypeScript-Handbook
