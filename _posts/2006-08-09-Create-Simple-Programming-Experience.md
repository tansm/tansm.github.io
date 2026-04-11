---
title: "创建简单的编程体验"
date: 2006-08-09 13:55
---
我始终认为，编程简单就是好，把复杂的问题简单化，模型尽可能的单一，这样才能创建良好的编程体验。

我希望编程应该是这样的：

## 使用方面

```csharp
ICreateService service = this.GetService<ICreateService>();

object data = service.Create();
```

使用方应该不关心服务的位置、创建方法等一系列过程。

## 创建方面

应该是面向方面的编程模型，每个类仅关注一个方面，而且应该是不关心是被谁调用的。他看起来应该非常的单纯。

```csharp
public class LoggerInterceptor {

    public void Log(string message) {

        //DO

    }

}
```

上下文信息应该是被注射进来的，可以非常方便的获取。

创建的过程应该是这样的：

![](/images/Create-Simple-Programming-Experience-1.png)

如果使用对象池中的对象，应该是直接到注射后调用。
