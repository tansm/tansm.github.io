# 转载：《TypeScript 中文入门教程》 11、声明合并

> 文章转载自：https://github.com/zhongsp

## 介绍

在 TypeScript 里，声明合并是指编译器会把两个相同名字的声明合并成一个单独的声明。合并后的声明同时具有那两个被合并的声明的特性。

## 基础概念

TypeScript 中的声明会创建以下三种实体之一：命名空间、类型或者值。

## 合并接口

最简单最常见的就是合并接口：

```typescript
interface Box {
    height: number;
    width: number;
}

interface Box {
    scale: number;
}

var box: Box = {height: 5, width: 6, scale: 10};
```

接口中非函数的成员必须是唯一的。对于函数成员，每个同名函数声明都会被当成这个函数的一个重载。

```typescript
interface Document {
    createElement(tagName: any): Element;
}
interface Document {
    createElement(tagName: string): HTMLElement;
}
interface Document {
    createElement(tagName: "div"): HTMLDivElement;
    createElement(tagName: "span"): HTMLSpanElement;
}
// 合并后，靠后的接口优先级更高
```

## 合并命名空间

同名的命名空间也会合并其成员：

```typescript
namespace Animals {
    export class Zebra { }
}

namespace Animals {
    export interface Legged { numberOfLegs: number; }
    export class Dog { }
}

// 等同于：
namespace Animals {
    export interface Legged { numberOfLegs: number; }
    export class Zebra { }
    export class Dog { }
}
```

非导出成员仅在其原始存在于的命名空间之内可见。

## 命名空间与类和函数和枚举类型合并

命名空间可以与其它类型的声明进行合并，实现内部类等模式：

```typescript
class Album {
    label: Album.AlbumLabel;
}
namespace Album {
    export class AlbumLabel { }
}
```

命名空间扩展函数：

```typescript
function buildLabel(name: string): string {
    return buildLabel.prefix + name + buildLabel.suffix;
}

namespace buildLabel {
    export var suffix = "";
    export var prefix = "Hello, ";
}
```

命名空间扩展枚举：

```typescript
enum Color {
    red = 1,
    green = 2,
    blue = 4
}

namespace Color {
    export function mixColor(colorName: string) {
        if (colorName == "yellow") {
            return Color.red + Color.green;
        }
        // ...
    }
}
```

## 非法的合并

类不能与类合并，变量与类型不能合并，接口与类不能合并。
