# 转载：《TypeScript 中文入门教程》 7、模块

> 文章转载自：https://github.com/zhongsp

## 介绍

从 ECMAScript 2015 开始，JavaScript 引入了模块的概念。TypeScript 也沿用这个概念。

模块在其自身的作用域里执行，而不是在全局作用域里。定义在一个模块里的变量、函数、类等在模块外部是不可见的，除非你明确地使用 `export` 导出它们。

## 导出

### 导出声明

任何声明都能够通过添加 `export` 关键字来导出：

```typescript
export interface StringValidator {
    isAcceptable(s: string): boolean;
}

export const numberRegexp = /^[0-9]+$/;

export class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
```

### 导出语句与重命名

```typescript
export { ZipCodeValidator };
export { ZipCodeValidator as mainValidator };
```

### 重新导出

```typescript
export { ZipCodeValidator as RegExpBasedZipCodeValidator } from "./ZipCodeValidator";
export * from "./StringValidator";
```

## 导入

```typescript
import { ZipCodeValidator } from "./ZipCodeValidator";
import { ZipCodeValidator as ZCV } from "./ZipCodeValidator";
import * as validator from "./ZipCodeValidator";
```

## 默认导出

每个模块都可以有一个 `default` 导出：

```typescript
// ZipCodeValidator.ts
export default class ZipCodeValidator {
    static numberRegexp = /^[0-9]+$/;
    isAcceptable(s: string) {
        return s.length === 5 && ZipCodeValidator.numberRegexp.test(s);
    }
}

// Test.ts
import validator from "./ZipCodeValidator";
var myValidator = new validator();
```

## `export =` 和 `import = require()`

TypeScript 模块支持 `export =` 语法，以配合传统的 CommonJS 和 AMD 的工作流：

```typescript
// ZipCodeValidator.ts
var numberRegexp = /^[0-9]+$/;
class ZipCodeValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
export = ZipCodeValidator;

// Test.ts
import zip = require("./ZipCodeValidator");
var validator = new zip.ZipCodeValidator();
```

## 创建模块结构指导

- 尽可能地在顶层导出，减少嵌套层次
- 如果仅导出单个 `class` 或 `function`，使用 `export default`
- 如果要导出多个对象，把它们放在顶层里导出
- 模块里不要使用命名空间（模块本身已经是一个逻辑分组）
