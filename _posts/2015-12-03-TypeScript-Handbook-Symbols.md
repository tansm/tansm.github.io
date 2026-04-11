# 转载：《TypeScript 中文入门教程》 16、Symbols

> 文章转载自：https://github.com/zhongsp

## 介绍

至 ECMAScript 2015 开始，`symbol` 成为了一种新的原始类型，就像 `number` 和 `string` 一样。

`symbol` 类型的值是通过 `Symbol` 构造函数创建的：

```typescript
var sym1 = Symbol();
var sym2 = Symbol("key"); // 可选的字符串 key
```

Symbols 是不可改变的且唯一的：

```typescript
var sym2 = Symbol("key");
var sym3 = Symbol("key");
sym2 === sym3; // false，symbols 是唯一的
```

像字符串一样，symbols 也可以被用做对象属性的键：

```typescript
let sym = Symbol();
let obj = {};
obj[sym] = "value";
console.log(obj[sym]); // "value"
```

Symbols 也可以与计算出的属性名声明相结合：

```typescript
const getClassNameSymbol = Symbol();

class C {
    [getClassNameSymbol](){
       return "C";
    }
}

let c = new C();
let className = c[getClassNameSymbol](); // "C"
```

## 内置 Symbols

除了用户定义的 symbols，还有一些已经熟悉的内置 symbols，用来表示语言内部的行为：

- `Symbol.hasInstance` - 被 `instanceof` 运算符调用
- `Symbol.isConcatSpreadable` - 表示数组元素是否可展开
- `Symbol.iterator` - 返回对象的默认迭代器，被 `for-of` 语句调用
- `Symbol.match` - 正则表达式方法用来匹配字符串，被 `String.prototype.match` 调用
- `Symbol.replace` - 被 `String.prototype.replace` 调用
- `Symbol.search` - 被 `String.prototype.search` 调用
- `Symbol.species` - 值为一个构造函数，用来创建对象实例
- `Symbol.split` - 被 `String.prototype.split` 调用
- `Symbol.toPrimitive` - 把对象转换为相应的原始值
- `Symbol.toStringTag` - 被 `Object.prototype.toString` 调用
- `Symbol.unscopables` - 被 `with` 作用域排除在外的属性
