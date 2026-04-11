# 使用 TypeScript 开发程序

## 简介

TypeScript 一直发展不错，我们公司在开发新功能时，考虑到程序的可维护性，使用了 TypeScript 编写浏览器上的程序，我们是从零开始使用 TypeScript，甚至我连 JavaScript 也是半罐子，本文描述了一个 C# 程序员认识 TypeScript 的过程。

注：本文编写时，基于 Typescript 0.8 版本，而且初用，可能过时，具体规范可以参考 http://www.typescriptlang.org

## 命名空间和类

作为面向对象的开发人员思维，第一个想到的是 TypeScript 如何定义类，由于正好我们项目服务端（C#）的原理和客户端（TypeScript）原理完全相同，所以这里正好用 C# 与 TypeScript 对比。

C# 声明类：

```csharp
using System;

namespace Digiwin.Mars.VirtualUI.Engine {
    internal sealed class Decoder {}
}
```

TypeScript 声明类：

```typescript
///<reference path="../Collections/ICollection.ts" />

module System.Erp.VirtualUI.Engine {
    export class Decoder {}
}
```

首先简单来说，都有类似命名空间的概念，一个叫 namespace，一个叫 module，这个就不废话了。

其次，C# 要引用其他的类，首先你需要在工程文件中引用 dll，然后在文件头上 using 一个命名空间（可选），但是在 TypeScript 中，没有这个概念，直接引用一个文件的。

C# 里类可以 public、internal 等等很多级别，还有 sealed 等修饰符，TypeScript 你就忘记这些吧，加入 export 相当于 public，抽象，值类型什么的，这个好像没有。但是接口是有的。

## 方法和注释

C# 的方法：

```csharp
/// <summary>
///   解码变更集
/// </summary>
/// <param name="reader"> 一个变更集读取器对象 </param>
public void DecodeChangeSet(ChangeRecordReader reader) {
    var ctx = new DecodeContext();
```

TypeScript 声明方法：

```typescript
/**
 * 传入变更集，将其解码到当前的对象容器。
 * @param {System.Erp.VirtualUI.Engine.IChangeRecordReader} reader - 提供记录集。
 */
public Decode(reader: IChangeRecordReader): void {
    var ctx = new DecodeContext();
```

TypeScript 使用 JsDoc 的规范。普通的注释也使用 `//`，这个完全和 JavaScript 相同。

在方法的声明上，TypeScript 将返回参数放在后面，对应的，参数的类型也是放在名字后面，如果你声明变量，也是这样的：

```typescript
private _maxId: number;  // 在类上定义字段
var item: VirtualObject; // 在方法里定义变量
```

在方法的可访问性上，支持 public，这样就可以公开还是不公开。

## 参数和构造

在 C# 里面，我们经常同一个名字定义多个方法，使用不同的参数类型区分，但是在 JavaScript 中不允许，所以 TypeScript 也不允许。由于上面的原因，你也就能理解只能有一个构造函数。下面是他的构造函数例子：

```typescript
constructor(
    objectContainer: VirtualObjectContainer,
    objectBinder: IObjectBinder
) {
    this._objectContainer = objectContainer;
    this._binder = objectBinder;
}
```

基于 JavaScript 的概念，也就没有 ref out in 这样的关键字，但有命名方式访问参数和可选参数。

好了，更多的细节需要你慢慢研究规范文档了，这篇文档可以帮助你入门，使用愉快。
