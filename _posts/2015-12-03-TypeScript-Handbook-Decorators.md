# 转载：《TypeScript 中文入门教程》 17、注解（Decorators）

> 文章转载自：https://github.com/zhongsp

## 介绍

随着 TypeScript 和 ES6 里引入了类，Decorators 提供了一种方式来添加注解和在类的声明和成员上使用元编程语法。

> 注意：Decorators 是实验性的特性，需要启用 `experimentalDecorators` 编译器选项。

```json
{
    "compilerOptions": {
        "target": "ES5",
        "experimentalDecorators": true
    }
}
```

## 装饰器

装饰器是一种特殊类型的声明，它能够被附加到类、方法、访问符、属性或参数上。装饰器利用 `@expression` 这种方式，`expression` 求值后必须为一个函数。

## 装饰器工厂

如果我们想自定义装饰器是如何作用于声明的，我们得写一个装饰器工厂函数：

```typescript
function color(value: string) { // 装饰器工厂
    return function (target) { // 装饰器
        // do something with "target" and "value"...
    }
}
```

## 装饰器组合

多个装饰器可以同时应用到一个声明上，求值方式与复合函数相似（由上至下求值，由下至上调用）：

```typescript
function f() {
    console.log("f(): evaluated");
    return function (target, propertyKey: string, descriptor: PropertyDescriptor) {
        console.log("f(): called");
    }
}

function g() {
    console.log("g(): evaluated");
    return function (target, propertyKey: string, descriptor: PropertyDescriptor) {
        console.log("g(): called");
    }
}

class C {
    @f()
    @g()
    method() {}
}
// 输出：f(): evaluated -> g(): evaluated -> g(): called -> f(): called
```

## 类装饰器

类装饰器在类声明之前被声明，应用于类构造函数：

```typescript
function sealed(constructor: Function) {
    Object.seal(constructor);
    Object.seal(constructor.prototype);
}

@sealed
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}
```

## 方法装饰器

方法装饰器声明在一个方法的声明之前，传入3个参数：目标对象、成员名字、成员的属性描述符：

```typescript
function enumerable(value: boolean) {
    return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
        descriptor.enumerable = value;
    };
}

class Greeter {
    @enumerable(false)
    greet() {
        return "Hello, " + this.greeting;
    }
}
```

## 访问符装饰器

```typescript
function configurable(value: boolean) {
    return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
        descriptor.configurable = value;
    };
}

class Point {
    @configurable(false)
    get x() { return this._x; }

    @configurable(false)
    get y() { return this._y; }
}
```

## 属性装饰器

属性装饰器表达式会在运行时当作函数被调用，传入2个参数：目标对象、成员名字。

## 参数装饰器

参数装饰器表达式会在运行时当作函数被调用，传入3个参数：目标对象、成员名字、参数在函数参数列表中的索引。

```typescript
class Greeter {
    @validate
    greet(@required name: string) {
        return "Hello " + name + ", " + this.greeting;
    }
}
```

## 元数据

TypeScript 支持为带有装饰器的声明生成元数据，需要启用 `emitDecoratorMetadata` 编译器选项，并安装 `reflect-metadata` 库：

```bash
npm i reflect-metadata --save
```
