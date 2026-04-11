---
title: "使用DependencyObject的例子"
date: 2006-08-07 14:45
---
在WinFX3.0整个UI的所有对象中，使用了DependencyObject对象，他简化了标准属性控制的流程。为XAML提供了基础支持

下面是使用这个对象的标准例子。

```
/// <summary>
    /// 使用DependencyObject的例子，定义了一个订单
    /// </summary>
    public class OrderSheet : DependencyObject {

        public static readonly DependencyProperty CodeProperty;

        /// <summary>
        /// 在静态构造中注册属性到此类型
        /// </summary>
        static OrderSheet() {
            CodeProperty = DependencyProperty.Register("Code", typeof(string), typeof(OrderSheet));
        }

        /// <summary>
        /// 返回/设置单据的编号
        /// </summary>
        public string Code {
            get {
                return (string)GetValue(CodeProperty);
            }
            set {
                SetValue(CodeProperty, value);
            }
        }
    }
```
