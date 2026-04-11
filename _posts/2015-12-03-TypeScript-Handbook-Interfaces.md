# 转载：《TypeScript 中文入门教程》 3、接口

> 文章转载自：https://github.com/zhongsp/TypeScript

## 介绍

TypeScript 的核心原则之一是对值所具有的 shape 进行类型检查，有时被称做"鸭式辨型法"或"结构性子类型化"。接口的作用就是为这些类型命名和为你的代码定义契约。

## 接口初探

```typescript
interface LabelledValue {
  label: string;
}

function printLabel(labelledObj: LabelledValue) {
  console.log(labelledObj.label);
}

var myObj = {size: 10, label: "Size 10 Object"};
printLabel(myObj);
```

类型检查器不会去检查属性的顺序，只要相应的属性存在并且类型也是对的就可以。

## 可选属性

接口里的属性不全都是必需的，可选属性在应用"option bags"模式时很常用：

```typescript
interface SquareConfig {
  color?: string;
  width?: number;
}

function createSquare(config: SquareConfig): {color: string; area: number} {
  var newSquare = {color: "white", area: 100};
  if (config.color) { newSquare.color = config.color; }
  if (config.width) { newSquare.area = config.width * config.width; }
  return newSquare;
}
```

## 函数类型

接口也可以描述函数类型，需要给接口定义一个调用签名：

```typescript
interface SearchFunc {
  (source: string, subString: string): boolean;
}

var mySearch: SearchFunc;
mySearch = function(src: string, sub: string): boolean {
  var result = src.search(sub);
  return result !== -1;
}
```

## 数组类型

```typescript
interface StringArray {
  [index: number]: string;
}
var myArray: StringArray = ["Bob", "Fred"];
```

## 类类型

与 C# 或 Java 里接口的基本作用一样，TypeScript 也能够用它来明确的强制一个类去符合某种契约：

```typescript
interface ClockInterface {
    currentTime: Date;
    setTime(d: Date);
}

class Clock implements ClockInterface {
    currentTime: Date;
    setTime(d: Date) { this.currentTime = d; }
    constructor(h: number, m: number) { }
}
```

## 扩展接口

接口可以相互扩展，一个接口可以继承多个接口：

```typescript
interface Shape { color: string; }
interface PenStroke { penWidth: number; }

interface Square extends Shape, PenStroke {
    sideLength: number;
}
```

## 混合类型

一个对象可以同时做为函数和对象使用，并带有额外的属性：

```typescript
interface Counter {
    (start: number): string;
    interval: number;
    reset(): void;
}
```

## 接口继承类

当接口继承了一个类类型时，它会继承类的成员但不包括其实现，同样会继承到类的 private 和 protected 成员。这意味着这个接口类型只能被这个类或其子类所实现。
