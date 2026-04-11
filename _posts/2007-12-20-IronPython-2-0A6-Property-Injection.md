---
layout: post
title: "使用IronPython 2.0A6 实现属性注入"
date: 2007-12-20 08:00:00 +0800
---
IronPython 1.0如何做属性注入很早就有朋友写了文章介绍了，请参考 《[IronPython 中的属性注入器机制](http://www.cnblogs.com/RChen/archive/2006/12/07/ipy_attr_inject.html) 》，这是[木野狐(Neil Chen) 的 Blog](http://www.cnblogs.com/RChen/) 兄的较详细的文章。

但是，到了IronPython 2.0这个程序完全变了麽样，在IronPython 2.0中包含了如何做属性注入的测试用例，为防止大家找的辛苦，我就将此代码放在此文章中便于大家搜索。注意本文章基于 2.0A6,其他版本（例如2.0A2）都有很大的出入。

下载IronPython 2.0A6源代码，你可以看见这个文件：Src\IronPythonTest\AttrInjectorTest.cs，他描述了如何帮助XmlElement实现属性注射。

首先你需要为整个组装件添加一个扩展标记：

```
[assembly: ExtensionType(typeof(XmlElement), typeof(IronPythonTest.AttrInjectorTest.SimpleXmlAttrInjector))]
```

第一个参数是你扩展的数据类型，第二个参数是扩展的切片类。下面是切片的代码：

```csharp
public class AttrInjectorTest {

        static AttrInjectorTest() {

            Microsoft.Scripting.RuntimeHelpers.RegisterAssembly(Assembly.GetExecutingAssembly());

        }

        public static class SimpleXmlAttrInjector {

            [SpecialName]

            public static IList<SymbolId> GetMemberNames(object obj) {

                List<SymbolId> list = new List<SymbolId>();

                XmlElement xml = obj as XmlElement;

                if (xml != null) {

                    for (XmlNode n = xml.FirstChild; n != null; n = n.NextSibling) {

                        if (n is XmlElement) {

                            list.Add(SymbolTable.StringToId(n.Name));

                        }

                    }

                }

                return list;

            }

            [SpecialName]

            public static object GetBoundMember(object obj, string name) {

                XmlElement xml = obj as XmlElement;

                if (xml != null) {

                    for (XmlNode n = xml.FirstChild; n != null; n = n.NextSibling) {

                        if (n is XmlElement && string.CompareOrdinal(n.Name, name) == 0) {

                            if (n.HasChildNodes && n.FirstChild == n.LastChild && n.FirstChild is XmlText) {

                                return n.InnerText;

                            } else {

                                return n;

                            }

                        }

                    }

                }

                return PythonOps.NotImplemented;

            }

        }

    }
```

可以看到，这个类包含了三个工作：

1、静态构造中包含了注册此组装件的方法，这样最初在组装件上标记的ExtensionTypeAttribute才真正生效；

2、GetMemberNames方法返回某个实例所有可用的扩展属性，非必须；

3、GetBoundMember方法才是关键，他返回某个扩展属性的值，在我自己的测试中发现，参数object obj是可以为强类型的，例如XmlElement。
