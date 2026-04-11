# 转载：《TypeScript 中文入门教程》 5、命名空间和模块

> 文章转载自：https://github.com/zhongsp

## 介绍

这篇文章将概括介绍在 TypeScript 里使用模块与命名空间来组织代码的方法。

## 使用命名空间

命名空间是位于全局命名空间下的一个普通的带有名字的 JavaScript 对象。它们可以在多文件中同时使用，并通过 `--outFile` 结合在一起。命名空间是帮你组织 Web 应用不错的方式，你可以把所有依赖都放在 HTML 页面的 `<script>` 标签里。

但就像其它的全局命名空间污染一样，它很难去了解组件之间的依赖关系，尤其是在大型的应用中。

## 使用模块

像命名空间一样，模块可以包含代码和声明。不同的是模块可以声明它的依赖。

模块会把依赖添加到模块加载器上（例如 CommonJs / Require.js）。对于大型应用，这一点点的花费会带来长久的模块化和可维护性上的便利。模块也提供了更好的代码重用，更强的封闭性以及更好的使用工具进行优化。

从 ECMAScript 2015 开始，模块成为了语言内置的部分，对于新项目来说推荐使用模块做为组织代码的方式。

## 命名空间和模块的陷阱

### 对模块使用 `/// <reference>`

一个常见的错误是使用 `/// <reference>` 引用模块文件，应该使用 `import`。

### 不必要的命名空间

如果你想把命名空间转换为模块，避免这样写：

```typescript
// shapes.ts - 错误示例
export namespace Shapes {
    export class Triangle { }
    export class Square { }
}
// 使用时：shapes.Shapes.Triangle - 多余的嵌套
```

应该直接导出：

```typescript
// shapes.ts - 正确示例
export class Triangle { }
export class Square { }
// 使用时：shapes.Triangle
```

TypeScript 里模块的一个特点是不同的模块永远也不会在相同的作用域内使用相同的名字，因此完全没有必要把导出的符号包裹在一个命名空间里。
