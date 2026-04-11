# 转载：《TypeScript 中文入门教程》 2、枚举

> 文章转载自：https://github.com/zhongsp/TypeScript/blob/master/doc/handbook/Enums.md

## 枚举

使用枚举我们可以定义一些有名字的数字常量。枚举通过使用 `enum` 关键字定义。

```typescript
enum Direction {
    Up = 1,
    Down,
    Left,
    Right
}
```

枚举成员被当成常数的条件：
- 不具有初始化函数并且前面的枚举成员是常数（第一个元素默认为 `0`）
- 枚举成员使用常数枚举表达式初始化

```typescript
enum FileAccess {
    None,
    Read    = 1 << 1,
    Write   = 1 << 2,
    ReadWrite  = Read | Write,
    G = "123".length  // computed member
}
```

枚举是在运行时真正存在的对象，支持从枚举值到枚举名的反射映射：

```typescript
enum Enum { A }
let a = Enum.A;
let nameOfA = Enum[Enum.A]; // "A"
```

编译后：

```javascript
var Enum;
(function (Enum) {
    Enum[Enum["A"] = 0] = "A";
})(Enum || (Enum = {}));
```

## 常数枚举

使用 `const` 修饰符可以定义常数枚举，在编译阶段会被删除，成员在使用处被内联：

```typescript
const enum Directions {
    Up, Down, Left, Right
}
let directions = [Directions.Up, Directions.Down, Directions.Left, Directions.Right]
```

编译后：

```javascript
var directions = [0 /* Up */, 1 /* Down */, 2 /* Left */, 3 /* Right */];
```

## 外部枚举

外部枚举用来描述已经存在的枚举类型的形状：

```typescript
declare enum Enum {
    A = 1,
    B,
    C = 2
}
```

外部枚举和非外部枚举的重要区别：在正常的枚举里，没有初始化方法的成员被当成常数成员；对于非常数的外部枚举而言，没有初始化方法时被当做需要经过计算的。
