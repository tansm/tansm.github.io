# 转载：《TypeScript 中文入门教程》 14、输入 .d.ts 文件

> 文章转载自：https://github.com/zhongsp

## 介绍

当使用外部 JavaScript 库或新的宿主 API 时，你需要一个声明文件（.d.ts）定义程序库的 shape。

## 指导与说明

### 流程

最好从程序库的文档开始写 .d.ts 文件，而不是代码。这样保证不会被具体实现所干扰，而且相比于 JS 代码更易读。

### 命名规则

一般来讲，不要给接口加 `I` 前缀（比如：`IColor`）。TypeScript 里的接口类型比 C# 或 Java 里的意义更为广泛，`IFoo` 命名不利于这个特点。

## 例子

### 参数对象

```typescript
// 使用方法
animalFactory.create("dog");
animalFactory.create("giraffe", { name: "ronald" });

// 类型声明
namespace animalFactory {
    interface AnimalOptions {
        name: string;
        height?: number;
        weight?: number;
    }
    function create(name: string, animalOptions?: AnimalOptions): Animal;
}
```

### 带属性的函数

```typescript
// 使用方法
zooKeeper.workSchedule = "morning";
zooKeeper(giraffeCage);

// 类型声明（函数必须在命名空间之前）
function zooKeeper(cage: AnimalCage): void;
namespace zooKeeper {
    var workSchedule: string;
}
```

### 可以用 new 调用也可以直接调用的方法

```typescript
// 使用方法
var w = widget(32, 16);
var y = new widget("sprocket");

// 类型声明
interface Widget {
    sprock(): void;
}

interface WidgetFactory {
    new(name: string): Widget;
    (width: number, height: number): Widget;
}

declare var widget: WidgetFactory;
```

### 全局的/不清楚的 Libraries

```typescript
// 类型声明
declare namespace zoo {
  function open(): void;
}

declare module "zoo" {
    export = zoo;
}
```

### 回调函数

```typescript
// 使用方法
addLater(3, 4, x => console.log('x = ' + x));

// 类型声明
function addLater(x: number, y: number, callback: (sum: number) => void): void;
```
