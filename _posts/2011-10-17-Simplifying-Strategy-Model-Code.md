# 简化策略模型的代码

> 原文：https://www.cnblogs.com/tansm/archive/2011/10/17/2215846.html

在我们编写代码时，经常遇到一次策略模式（俺不会背那个啥设计模式，暂时叫他策略模式吧），例如，在反序列化时，已知一个名称和命名空间，获取其对应的类型，使用下面的策略：

尝试从绑定期中获取，

如果不成功，尝试从基类获取；

如果还不成功，尝试播发事件获取。

看起来，一个个尝试，如果不成功，下一个。代码是这个样子的。

```text
private IEntityType BindToType(XElement element,IEntityType baseType, out IEnumerable<DcxmlBinder.CustomAttribute> attributes)
        {
            //第一级策略
            var name = element.Name;
            attributes = from p in element.Attributes()
                         select new DcxmlBinder.CustomAttribute() { Namespace = p.Name.NamespaceName, Name = p.Name.LocalName, Value = p.Value };
            IEntityType dt = _binder.BindToType(name.NamespaceName, name.LocalName,attributes);

            if (dt == null)
            {
                //第二级策略:如果没有，看看是否有缺省类型。
                if (baseType != null)
                {
                    //检查此类型的确正好是此名称
                    string n1, n2;
                    _binder.BindToName(baseType, out n1, out n2);
                    if (string.Equals(name.NamespaceName, n1, _binder.StringComparison) &&
                        string.Equals(name.LocalName, n2, _binder.StringComparison))
                    {
                        //不用检测派生关系，直接返回。
                        return baseType;
                    }
                }

                //第三级策略
                //TODO:应该先播发event System.Xml.Serialization.XmlElementEventHandler UnknownElement事件。外界获取类型。
                throw new ApplicationException();
            }

            //检查派生关系是否正确，即dt必须是baseDt的派生类型。
            if ((baseType != null) && (!baseType.IsAssignableFrom(dt)))
            {
                throw new ApplicationException("设计错误，返回的数据类型没有派生自baseType");
            }

            return dt;
        }
```

　　

有点乱糟糟的，老实说，我也觉得乱糟糟的。于是我也想优化这样的代码，最终，我利用了C# 的或短路的特性，代码如下：

```text
private IEntityType BindToType(XElement element,IEntityType baseType, out IEnumerable<DcxmlBinder.CustomAttribute> attributes)
        {
            IEntityType dt;
            if (TryBindFromBinder(element,out dt,out attributes) ||
                TryBindFromBase(element,baseType,out dt) ||
                TryBindFromEvent(element,baseType,out dt))
            {
                //检查派生关系是否正确，即dt必须是baseDt的派生类型。
                if ((baseType != null) && (!baseType.IsAssignableFrom(dt)))
                {
                    throw new ApplicationException("设计错误，返回的数据类型没有派生自baseType");
                }
            }

            return dt;
        }

        private bool TryBindFromBinder(XElement element, out IEntityType dt, out IEnumerable<DcxmlBinder.CustomAttribute> attributes)
        {
            //第一级策略
            var name = element.Name;
            attributes = from p in element.Attributes()
                         select new DcxmlBinder.CustomAttribute() { Namespace = p.Name.NamespaceName, Name = p.Name.LocalName, Value = p.Value };
            dt = _binder.BindToType(name.NamespaceName, name.LocalName,attributes);

            return (dt != null);
        }

        private bool TryBindFromBase(XElement element, IEntityType baseType, out IEntityType dt)
        {
            //第二级策略:如果没有，看看是否有缺省类型。
            if (baseType != null)
            {
                var name = element.Name;
                //检查此类型的确正好是此名称
                string n1, n2;
                _binder.BindToName(baseType, out n1, out n2);
                if (string.Equals(name.NamespaceName, n1, _binder.StringComparison) &&
                    string.Equals(name.LocalName, n2, _binder.StringComparison))
                {
                    //不用检测派生关系，直接返回。
                    dt = baseType;
                    return true;
                }
            }

            dt = null;
            return false;
        }

        private bool TryBindFromEvent(XElement element, IEntityType baseType, out IEntityType dt)
        {
            //第三级策略
            //TODO:应该先播发event System.Xml.Serialization.XmlElementEventHandler UnknownElement事件。外界获取类型。
            throw new ApplicationException();
        }
```

　　虽然代码比以前多了，但是看起来清晰了很多。而且，通常第一个策略都能完成搜索，所以就不会调用后面的方法，也就不会初始化其变量了。
