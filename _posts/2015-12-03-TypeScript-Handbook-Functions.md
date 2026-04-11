# 转载：《TypeScript 中文入门教程》 8、函数

> 文章转载自：https://github.com/zhongsp

## 介绍

函数是 JavaScript 应用程序的基础。TypeScript 为 JavaScript 函数添加了额外的功能，让我们可以更容易的使用。

## 函数类型

### 为函数定义类型

```typescript
function add(x: number, y: number): number {
    return x + y;
}

var myAdd = function(x: number, y: number): number { return x + y; };
```

### 书写完整函数类型

```typescript
var myAdd: (x: number, y: number) => number =
    function(x: number, y: number): number { return x + y; };
```

## 可选参数和默认参数

TypeScript 里的每个函数参数都是必须的。可以在参数名旁使用 `?` 实现可选参数：

```typescript
function buildName(firstName: string, lastName?: string) {
    if (lastName)
        return firstName + " " + lastName;
    else
        return firstName;
}
```

也可以为参数提供一个默认值：

```typescript
function buildName(firstName: string, lastName = "Smith") {
    return firstName + " " + lastName;
}
```

## 剩余参数

```typescript
function buildName(firstName: string, ...restOfName: string[]) {
    return firstName + " " + restOfName.join(" ");
}

var employeeName = buildName("Joseph", "Samuel", "Lucas", "MacKinzie");
```

## Lambda 表达式和 `this`

JavaScript 里 `this` 的值在函数被调用的时候才会指定。使用 lambda 表达式（箭头函数）可以在函数创建的时候就指定 `this` 值：

```typescript
var deck = {
    suits: ["hearts", "spades", "clubs", "diamonds"],
    cards: Array(52),
    createCardPicker: function() {
        return () => {  // 使用箭头函数捕获 this
            var pickedCard = Math.floor(Math.random() * 52);
            var pickedSuit = Math.floor(pickedCard / 13);
            return {suit: this.suits[pickedSuit], card: pickedCard % 13};
        }
    }
}
```

## 重载

为同一个函数提供多个函数类型定义来进行函数重载：

```typescript
function pickCard(x: {suit: string; card: number; }[]): number;
function pickCard(x: number): {suit: string; card: number; };
function pickCard(x): any {
    if (typeof x == "object") {
        var pickedCard = Math.floor(Math.random() * x.length);
        return pickedCard;
    }
    else if (typeof x == "number") {
        var pickedSuit = Math.floor(x / 13);
        return { suit: suits[pickedSuit], card: x % 13 };
    }
}
```

注意：在定义重载的时候，一定要把最精确的定义放在最前面。
