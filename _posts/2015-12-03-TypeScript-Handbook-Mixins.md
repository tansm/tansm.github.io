# 转载：《TypeScript 中文入门教程》 10、混入

> 文章转载自：https://github.com/zhongsp

## 介绍

除了传统的面向对象继承方式，还流行一种通过可重用组件创建类的方式，就是联合另一个简单类的代码。你可能在 Scala 等语言里对 mixins 及其特性已经很熟悉了，但它在 JavaScript 中也是很流行的。

## 混入示例

```typescript
// Disposable Mixin
class Disposable {
    isDisposed: boolean;
    dispose() {
        this.isDisposed = true;
    }
}

// Activatable Mixin
class Activatable {
    isActive: boolean;
    activate() {
        this.isActive = true;
    }
    deactivate() {
        this.isActive = false;
    }
}

class SmartObject implements Disposable, Activatable {
    constructor() {
        setInterval(() => console.log(this.isActive + " : " + this.isDisposed), 500);
    }

    interact() {
        this.activate();
    }

    // Disposable 占位属性
    isDisposed: boolean = false;
    dispose: () => void;
    // Activatable 占位属性
    isActive: boolean = false;
    activate: () => void;
    deactivate: () => void;
}
applyMixins(SmartObject, [Disposable, Activatable]);

var smartObj = new SmartObject();
setTimeout(() => smartObj.interact(), 1000);

// 帮助函数
function applyMixins(derivedCtor: any, baseCtors: any[]) {
    baseCtors.forEach(baseCtor => {
        Object.getOwnPropertyNames(baseCtor.prototype).forEach(name => {
            derivedCtor.prototype[name] = baseCtor.prototype[name];
        });
    });
}
```

## 理解这个例子

首先定义了两个类，它们将做为 mixins，每个类都只定义了一个特定的行为或功能。

关键点：使用 `implements` 而不是 `extends`，把类当成了接口，仅使用 Disposable 和 Activatable 的类型而非其实现。

然后为将要 mixin 进来的属性方法创建出占位属性，告诉编译器这些成员在运行时是可用的。

最后，通过 `applyMixins` 帮助函数，遍历 mixins 上的所有属性，并复制到目标上去，把之前的占位属性替换成真正的实现代码。
