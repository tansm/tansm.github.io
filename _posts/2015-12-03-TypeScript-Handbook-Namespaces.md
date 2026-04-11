# 转载：《TypeScript 中文入门教程》 6、命名空间

> 文章转载自：https://github.com/zhongsp
>
> 注意：TypeScript 1.5 里术语名已经发生了变化。"内部模块"现在称做"命名空间"。"外部模块"现在则简称为"模块"。

## 介绍

这篇文章描述了如何在 TypeScript 里使用命名空间（之前叫做"内部模块"）来组织你的代码。

## 第一步：所有验证器放在一个文件里

```typescript
interface StringValidator {
    isAcceptable(s: string): boolean;
}

var lettersRegexp = /^[A-Za-z]+$/;
var numberRegexp = /^[0-9]+$/;

class LettersOnlyValidator implements StringValidator {
    isAcceptable(s: string) {
        return lettersRegexp.test(s);
    }
}

class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
```

## 使用命名空间

把验证器包裹到一个命名空间内，避免全局命名冲突：

```typescript
namespace Validation {
    export interface StringValidator {
        isAcceptable(s: string): boolean;
    }

    const lettersRegexp = /^[A-Za-z]+$/;
    const numberRegexp = /^[0-9]+$/;

    export class LettersOnlyValidator implements StringValidator {
        isAcceptable(s: string) {
            return lettersRegexp.test(s);
        }
    }

    export class ZipCodeValidator implements StringValidator {
        isAcceptable(s: string) {
            return s.length === 5 && numberRegexp.test(s);
        }
    }
}

var validators: { [s: string]: Validation.StringValidator; } = {};
validators["ZIP code"] = new Validation.ZipCodeValidator();
validators["Letters only"] = new Validation.LettersOnlyValidator();
```

## 分离到多文件

当应用变得越来越大时，我们需要将代码分离到不同的文件中以便于维护。尽管是不同的文件，它们仍是同一个命名空间，并且在使用的时候就如同它们在一个文件中定义的一样。

```typescript
// Validation.ts
namespace Validation {
    export interface StringValidator {
        isAcceptable(s: string): boolean;
    }
}

// LettersOnlyValidator.ts
/// <reference path="Validation.ts" />
namespace Validation {
    const lettersRegexp = /^[A-Za-z]+$/;
    export class LettersOnlyValidator implements StringValidator {
        isAcceptable(s: string) {
            return lettersRegexp.test(s);
        }
    }
}
```

## 别名

使用 `import q = x.y.z` 给常用的对象起一个短的名字：

```typescript
namespace Shapes {
    export namespace Polygons {
        export class Triangle { }
        export class Square { }
    }
}

import polygons = Shapes.Polygons;
var sq = new polygons.Square(); // Same as "new Shapes.Polygons.Square()"
```

## 外部命名空间

为第三方库声明类型时，使用外部命名空间：

```typescript
declare namespace D3 {
    export interface Selectors {
        select: {
            (selector: string): Selection;
            (element: EventTarget): Selection;
        };
    }
    export interface Base extends Selectors {
        event: Event;
    }
}

declare var d3: D3.Base;
```
