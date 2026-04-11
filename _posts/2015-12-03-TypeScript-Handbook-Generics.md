# 转载：《TypeScript 中文入门教程》 9、泛型

> 文章转载自：https://github.com/zhongsp

## 介绍

软件工程中，我们不仅要创建一致的定义良好的 API，同时也要考虑可重用性。泛型可以让组件支持多种类型的数据，这样用户就可以以自己的数据类型来使用组件。

## 泛型之 Hello World

不用泛型的话，函数可能是这样：

```typescript
function identity(arg: number): number {
    return arg;
}
// 或者使用 any，但会丢失类型信息
function identity(arg: any): any {
    return arg;
}
```

使用类型变量 `T`，它是一种特殊的变量，只用于表示类型而不是值：

```typescript
function identity<T>(arg: T): T {
    return arg;
}

// 两种调用方式
var output = identity<string>("myString");  // 明确指定类型
var output = identity("myString");          // 类型推论
```

## 使用泛型变量

```typescript
function loggingIdentity<T>(arg: T[]): T[] {
    console.log(arg.length);  // Array has .length
    return arg;
}
// 等价写法
function loggingIdentity<T>(arg: Array<T>): Array<T> {
    console.log(arg.length);
    return arg;
}
```

## 泛型接口

```typescript
interface GenericIdentityFn<T> {
    (arg: T): T;
}

function identity<T>(arg: T): T {
    return arg;
}

var myIdentity: GenericIdentityFn<number> = identity;
```

## 泛型类

```typescript
class GenericNumber<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}

var myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x, y) { return x + y; };
```

## 泛型约束

使用 `extends` 关键字来约束泛型：

```typescript
interface Lengthwise {
    length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);  // 现在知道有 .length 属性
    return arg;
}

loggingIdentity(3);  // Error, number 没有 .length 属性
loggingIdentity({length: 10, value: 3});  // OK
```

## 在泛型里使用类类型

在 TypeScript 使用泛型创建工厂函数时，需要引用构造函数的类类型：

```typescript
function create<T>(c: {new(): T; }): T {
    return new c();
}
```
