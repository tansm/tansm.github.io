---
title: "从校验功能浅谈重构"
date: 2009-09-15 00:00:00 +0800
---

d其实不知道用啥标题，因为这篇文章纯粹的“随笔”。因为今天cnblogs上看见两篇关于校验的文章，《[Smark.Data实体成员数据验证](http://www.cnblogs.com/henryfan/archive/2009/09/15/1566951.html)》和《[浅谈Oxite2的数据访问和数据验证](http://www.cnblogs.com/xuefly/archive/2009/09/14/1566545.html)》，加上《[代码习惯](http://www.cnblogs.com/wangyh/archive/2009/09/15/clean-code.html)》这篇文章有些感触，所以就着这两个东西写点模式应用，没有任何贬低两个框架的意思，我只关注技术。
讲的其实都是非常简单的东西，主要受《代码习惯》文章的启发，觉得很多时候道理大家都知道，但是没有去严格要求自己，我这里只是强调一下。

1、单一职责
我们好多代码喜欢把功能堆砌在一起，这个我们就需要拆分开，例如《[Smark.Data实体成员数据验证](http://www.cnblogs.com/henryfan/archive/2009/09/15/1566951.html)》一文中（您可能需要移步跳转到那篇文章，看一下代码），其红色的代码部分包含了校验功能，但是这个函数却是：NewData，看样子是做插入操作的，因此我不建议此函数中包含具体的校验实现。这个时候我们应该重构他，让这些功能移动到某个类，最好是某个服务中，这样让NewData更加单一职责。
在第二篇文章《[浅谈Oxite2的数据访问和数据验证](http://www.cnblogs.com/xuefly/archive/2009/09/14/1566545.html)》中提及的Oxite2框架中，我看了其[代码的实现](http://oxite.codeplex.com/sourcecontrol/changeset/view/43272?projectName=oxite#387618)，其就定义了IValidationService接口。
```text
    public interface IValidationService
    {
        ValidationState Validate(T model);
    }
```

**一个函数甚至一个类就做一类事情**。道理很简单，我就不啰嗦了。2、简单设计这回拿Oxite2说事，咱先来看其校验服务的默认实现：

```text
    public class ValidationService : IValidationService
    {
        private readonly IUnityContainer container;

        public ValidationService(IUnityContainer container)
        {
            this.container = container;
        }

        public IValidator GetValidatorFor(T entity)
        {
            return container.Resolve>();
        }

        public ValidationState Validate(T entity)
        {
            IValidator validator = GetValidatorFor(entity);

            if (validator == null) // or just return null?
                throw new Exception(string.Format("No validator found for type ({0})", entity.GetType()));

            return validator.Validate(entity);
        }
    }
```

很容易理解其代码的用意，注意其GetValidatorFor的实现，我们会发现他从容器中获取一个校验器实例，而且是唯一的实例。这个问题就来了：为什么我需要IValidator？IValidationService是负责校验的，IValidator也是负责校验的，两个职责重叠了。现在的代码是先调用校验服务，然后校验服务转发给具体实体类型的校验器。我觉得完全可以删除IValidator，修改为IValidationService。后话：我注意到IValidationService算是全局服务，非泛型的，而IValidator是泛型的，针对每种类型的实体。作者意图可能希望起到一个转发的作用，但是我认为，既然可以配置IValidator，一样可以配置出IValidationService。**很多程序现在热衷于各种设计模式，讲指令转发过来转发过去，其实很多时候简单才好。**3、简单细节既然是随笔，就没有那么多的1234了，看到哪里就说到哪里。下面是Oxite2中错误的代码：

```text
        public ValidationState()
        {
            Errors = new ValidationErrorCollection();
        }

        public ValidationState(IEnumerable errors)
            : this()
        {
            if (errors != null)
                foreach (ValidationError error in errors)
                    Errors.Add(error);
        }
```这个代码有问题，在构造函数中，**应该是参数少的调用参数多的，如果需要调用基类的构造，应该让参数最多的构造调用基类参数最多的构造**。
一来和方法的调用规则一致，二来不用看代码跳来跳去。下面是正确的代码：

```text
        public ValidationState() : this(null)
        {
        }

        public ValidationState(IEnumerable errors)
        {
            this.Errors = new ValidationErrorCollection();
            if (errors != null)
            {
                foreach (ValidationError error in errors)
                    Errors.Add(error);
            }
        }
```
还有一个地方就是封装集合类的问题，我的观点是，如果你定义了集合类，一般就应该具有封装性，简单的说就是不要派生自List或Dictionary，不然你封装他干嘛？
下面是Oxite2中错误的代码，他错误的从List上继承。

```text
public class ValidationErrorCollection : List
{
    public void Add(string name, object attemptedValue, string message)
    {
        Add(new ValidationError(name, attemptedValue, message));
    }

    public void Add(string name, object attemptedValue, Exception exception)
    {
        Add(new ValidationError(name, attemptedValue, exception));
    }
}
```**正确的方法是从Collection上派生，需要注意的是Collection是允许插入null的，所以请记得重载插入操作禁止插入null。**

**最后就是注释了**，Oxite2注释好少啊。
