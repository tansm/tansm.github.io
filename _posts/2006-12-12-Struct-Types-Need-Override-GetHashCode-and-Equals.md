---
layout: post
title: "结构类型需要重载GetHashCode和Equals"
date: 2006-12-12 08:00:00 +0800
---

> 原文：https://www.cnblogs.com/tansm/archive/2006/12/12/589569.html
我比较迟钝，到现在才知道结构类型放在字典作为键时，效率是不好的，最好重载GetHashCode和Equals，那效率究竟差异有多大呢？我写了一个测试程序。

```text
ResourceServiceKey2 key1 = new ResourceServiceKey2(typeof(IRegisterAssemblyService),"OrderSheet");
           ResourceServiceKey2 key2 = new ResourceServiceKey2(typeof(IRegisterAssemblyService), "OrderSheet");
           DateTime begin = DateTime.Now;
           DateTime endTime;           bool r;
           for (int i = 0; i < 10000000; i++) {               r = key1.Equals(key2);
           }           endTime = DateTime.Now;
           TimeSpan s = begin - endTime;
           ResourceServiceKey key3 = new ResourceServiceKey(typeof(IRegisterAssemblyService), "OrderSheet");
           ResourceServiceKey key4 = new ResourceServiceKey(typeof(IRegisterAssemblyService), "OrderSheet");
           begin = DateTime.Now;
           for (int i = 0; i < 10000000; i++) {               r = key3.Equals(key4);
           }
           endTime = DateTime.Now;
          TimeSpan s2 = begin - endTime;
```

ResourceServiceKey是重载了这两个方法的，共花了0.5秒左右，而ResourceServiceKey2没有重载，共花了22秒，差异还是挺大的。反编译ValueType（结构的基础类型），看了代码，才发现，这样不慢才怪呢。

```csharp
public override bool Equals(object obj)
{
if (obj == null)
{
return false;
}
RuntimeType type1 = (RuntimeType) base.GetType();
RuntimeType type2 = (RuntimeType) obj.GetType();
if (type2 != type1)
{
return false;
}
object obj1 = this;
if (ValueType.CanCompareBits(this))
{
return ValueType.FastEqualsCheck(obj1, obj);
}
FieldInfo[] infoArray1 = type1.GetFields(BindingFlags.NonPublic | BindingFlags.Public | BindingFlags.Instance);
for (int num1 = 0; num1 < infoArray1.Length; num1++)
{
object obj2 = ((RtFieldInfo) infoArray1[num1]).InternalGetValue(obj1, false);
object obj3 = ((RtFieldInfo) infoArray1[num1]).InternalGetValue(obj, false);
if (obj2 == null)
{
if (obj3 != null)
{
return false;
}
}
else if (!obj2.Equals(obj3))
{
return false;
}
}
return true;
}
```
