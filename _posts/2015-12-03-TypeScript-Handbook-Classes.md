# 转载：《TypeScript 中文入门教程》 4、类

> 文章转载自：https://zhongsp.gitbooks.io/typescript-handbook/content/doc/handbook/Classes.html

## 介绍

从 ECMAScript 2015 开始，JavaScript 程序可以使用基于类的面向对象方法。在 TypeScript 里，我们允许开发者现在就使用这些特性，并且编译后的 JavaScript 可以在所有主流浏览器和平台上运行。

## 类

```typescript
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}

var greeter = new Greeter("world");
```

## 继承

```typescript
class Animal {
    name: string;
    constructor(theName: string) { this.name = theName; }
    move(distanceInMeters: number = 0) {
        console.log(`${this.name} moved ${distanceInMeters}m.`);
    }
}

class Snake extends Animal {
    constructor(name: string) { super(name); }
    move(distanceInMeters = 5) {
        console.log("Slithering...");
        super.move(distanceInMeters);
    }
}
```

包含 constructor 函数的派生类必须调用 `super()`，它会执行基类的构造方法。

## 公共、私有与受保护的修饰符

在 TypeScript 里，每个成员默认为 `public`。

```typescript
class Animal {
    private name: string;
    constructor(theName: string) { this.name = theName; }
}
new Animal("Cat").name; // Error: 'name' is private
```

`protected` 修饰符与 `private` 修饰符的行为很相似，但 `protected` 成员在派生类中仍然可以访问。

## 参数属性

参数属性可以方便地让我们在一个地方定义并初始化一个成员：

```typescript
class Animal {
    constructor(private name: string) { }
    move(distanceInMeters: number) {
        console.log(`${this.name} moved ${distanceInMeters}m.`);
    }
}
```

## 存取器

TypeScript 支持 getters/setters 来截取对对象成员的访问：

```typescript
class Employee {
    private _fullName: string;

    get fullName(): string { return this._fullName; }

    set fullName(newName: string) {
        if (passcode && passcode == "secret passcode") {
            this._fullName = newName;
        } else {
            console.log("Error: Unauthorized update of employee!");
        }
    }
}
```

注意：若要使用存取器，要求设置编译器输出目标为 ECMAScript 5 或更高。

## 静态属性

```typescript
class Grid {
    static origin = {x: 0, y: 0};
    calculateDistanceFromOrigin(point: {x: number; y: number;}) {
        var xDist = (point.x - Grid.origin.x);
        var yDist = (point.y - Grid.origin.y);
        return Math.sqrt(xDist * xDist + yDist * yDist) / this.scale;
    }
    constructor (public scale: number) { }
}
```

## 抽象类

抽象类是供其它类继承的基类，一般不会直接被实例化：

```typescript
abstract class Animal {
    abstract makeSound(): void;
    move(): void {
        console.log('roaming the earth...');
    }
}
```

## 把类当做接口使用

```typescript
class Point {
    x: number;
    y: number;
}

interface Point3d extends Point {
    z: number;
}

var point3d: Point3d = {x: 1, y: 2, z: 3};
```
