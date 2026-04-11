# 转载：《TypeScript 中文入门教程》 15、可迭代性

> 文章转载自：https://github.com/zhongsp

## 可迭代性

当一个对象实现了 `Symbol.iterator` 属性时，我们认为它是可迭代的。一些内置的类型如 `Array`、`Map`、`Set`、`String`、`Int32Array`、`Uint32Array` 等都已经实现了各自的 `Symbol.iterator`。

## `for..of` 语句

`for..of` 会遍历可迭代的对象，调用对象上的 `Symbol.iterator` 属性：

```typescript
let someArray = [1, "string", false];

for (let entry of someArray) {
    console.log(entry); // 1, "string", false
}
```

## `for..of` vs. `for..in` 语句

`for..of` 和 `for..in` 均可迭代一个列表，但用于迭代的值却不同：
- `for..in` 迭代的是对象的**键**的列表
- `for..of` 则迭代对象数字键对应的**值**

```typescript
let list = [4, 5, 6];

for (let i in list) {
   console.log(i); // "0", "1", "2"
}

for (let i of list) {
   console.log(i); // 4, 5, 6
}
```

`Map` 和 `Set` 已经实现了 `Symbol.iterator` 属性：

```typescript
let pets = new Set(["Cat", "Dog", "Hamster"]);
pets["species"] = "mammals";

for (let pet in pets) {
   console.log(pet); // "species"
}

for (let pet of pets) {
    console.log(pet); // "Cat", "Dog", "Hamster"
}
```

## 代码生成

当生成目标为 ES5 或 ES3 时，迭代器只允许在 `Array` 类型上使用。编译器会生成一个简单的 `for` 循环：

```typescript
// 源代码
let numbers = [1, 2, 3];
for (let num of numbers) {
    console.log(num);
}

// 编译后
var numbers = [1, 2, 3];
for (var _i = 0; _i < numbers.length; _i++) {
    var num = numbers[_i];
    console.log(num);
}
```

当目标为兼容 ECMAScript 2015 的引擎时，编译器会生成相应引擎的 `for..of` 内置迭代器实现方式。
