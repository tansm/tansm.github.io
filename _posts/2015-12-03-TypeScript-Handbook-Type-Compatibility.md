# 转载：《TypeScript 中文入门教程》 13、类型兼容性

> 文章转载自：https://github.com/zhongsp

## 介绍

TypeScript 里的类型兼容性基于结构子类型的。结构类型是只使用其成员来描述类型的方式，它正好与名义类型形成对比：

```typescript
interface Named {
    name: string;
}

class Person {
    name: string;
}

var p: Named;
p = new Person();  // OK，因为结构类型兼容
```

在使用名义类型的语言（比如 C# 或 Java）中，这段代码会报错，因为 Person 类没有明确说明其实现了 Named 接口。

## 开始

TypeScript 结构化类型系统的基本规则是，如果 `x` 要兼容 `y`，那么 `y` 至少具有与 `x` 相同的属性：

```typescript
interface Named { name: string; }

var x: Named;
var y = { name: 'Alice', location: 'Seattle' };
x = y;  // OK，y 包含 name 属性
```

## 比较两个函数

```typescript
var x = (a: number) => 0;
var y = (b: number, s: string) => 0;

y = x;  // OK，x 的参数在 y 中都能找到
x = y;  // Error，y 有必需的第二个参数，x 没有
```

JavaScript 里忽略额外的参数是很常见的（如 `Array#forEach` 的回调），TypeScript 允许这种行为。

返回值类型：

```typescript
var x = () => ({name: 'Alice'});
var y = () => ({name: 'Alice', location: 'Seattle'});

x = y;  // OK
y = x;  // Error，x() 缺少 location 属性
```

## 枚举

枚举类型与数字类型兼容，并且数字类型与枚举类型兼容。不同枚举类型之间是不兼容的：

```typescript
enum Status { Ready, Waiting };
enum Color { Red, Blue, Green };

var status = Status.Ready;
status = Color.Green;  // Error
```

## 类

类与对象字面量和接口差不多，但有一点不同：类有静态部分和实例部分的类型。比较两个类类型的对象时，只有实例的成员会被比较，静态成员和构造函数不在比较的范围内：

```typescript
class Animal {
    feet: number;
    constructor(name: string, numFeet: number) { }
}

class Size {
    feet: number;
    constructor(numFeet: number) { }
}

var a: Animal;
var s: Size;

a = s;  // OK
s = a;  // OK
```

私有成员会影响兼容性判断：如果目标类型包含一个私有成员，那么源类型必须包含来自同一个类的这个私有成员。

## 泛型

因为 TypeScript 是结构性的类型系统，类型参数只影响使用其做为类型一部分的结果类型：

```typescript
interface Empty<T> { }
var x: Empty<number>;
var y: Empty<string>;
x = y;  // OK，结构相同

interface NotEmpty<T> { data: T; }
var x: NotEmpty<number>;
var y: NotEmpty<string>;
x = y;  // Error，结构不同
```
