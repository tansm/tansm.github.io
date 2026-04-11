# 为CLR对象动态添加属性

> 原文：https://www.cnblogs.com/tansm/archive/2010/07/08/1773888.html

在脚本语言的世界里，你可以在运行时随意将任何一个对象添加一个属性，例如：  
obj.MyData = 1;  
鬼才知道obj原先有没有MyData属性呢，在脚本引擎执行时，他会为这个对象动态添加一个属性MyData，以后你就可以访问它了，而且你需要注意哦：添加的属性依附在实例上，而不是类型上。  
好了，言归正传，在CLR的世界中，我们有样的好东东吗？答案是：有！当然有，DLR基于CLR编写的，最么可能他自己没有呢。  
这要感谢.net 4新提供的ConditionalWeakTable<TKey, TValue>类,MSDN是这么解释的：  
   
ConditionalWeakTable<TKey, TValue> 类使得语言编译器可在运行时给托管的对象附加任意属性。ConditionalWeakTable<TKey, TValue> 对象是一个字典，它将一个键表示的托管对象绑定到一个值表示的附加属性。该对象的键是 TKey 类的个体实例，属性附加到这个类上，且其值为分配到相应对象的属性值。  
   
有点拗口，没有关系，看代码：

```csharp
using System; 
using System.Collections.Generic; 
using System.Linq; 
using System.Text; 
using System.Runtime.CompilerServices; 

namespace ConditionalWeakTableDemo 
{ 
    class Program 
    { 
        static void Main(string[] args) 
        { 
            //演示 为某个对象动态添加属性。 
            object a1 = new object(); 
            object a2 = new object(); 

            a1.SetAttachedPropertyValue<string>("Name", "a1"); 
            a2.SetAttachedPropertyValue<string>("Name", "a2"); 

            Console.WriteLine("(a1)my name is " + a1.GetAttachedPropertyValue<string>("Name")); 
            Console.WriteLine("(a2)my name is " + a2.GetAttachedPropertyValue<string>("Name")); 

            //观察 内存变化，看是否有内存泄漏 
            for (int i = 0; i < 1000000000; i++) 
            { 
                a1 = new object(); 
                a1.SetAttachedPropertyValue<int>("Number", i); 
            } 
        } 
    } 

    public static class AttachedProperty 
    { 
        private static ConditionalWeakTable<object, Dictionary<string, object>> _table = 
             new ConditionalWeakTable<object, Dictionary<string, object>>(); 

        public static T GetAttachedPropertyValue<T>(this object owner,string propertyName) 
        { 
            Dictionary<string, object> values; 
            if (_table.TryGetValue(owner,out values)) 
            { 
                object temp; 
                if (values.TryGetValue(propertyName,out temp)) 
                { 
                    return (T)temp; 
                } 
            } 

            return default(T); 
        } 

        public static void SetAttachedPropertyValue<T>(this object owner, string propertyName, T value) 
        { 
            Dictionary<string, object> values; 
            if (!_table.TryGetValue(owner, out values)) 
            { 
                values = new Dictionary<string, object>(); 
                _table.Add(owner, values); 
            } 

            values[propertyName] = value; 
        } 
    } 
}
```

代码不算长，仔细看看就明白了，我们封装了ConditionalWeakTable，这样就可以为任何实例附加一个属性了。

当然，如果能做到这样就最好了：

```csharp
dynamic a1 = new Object();
a1.Name = "tansm";
```

 谁能搞定？
