---
title: "今天写的代码，可惜没有用上，但想想可能以后还能用上吧。"
date: 2008-03-10 00:00:00 +0800
---

# 今天写的代码，可惜没有用上，但想想可能以后还能用上吧。

是关于自定义数据的简单实现，你可以这样使用。

```csharp
//define ConnectionString property.

CustomProperty<string> ConnectionStringProperty =

    new CustomProperty<string>("ConnectionStringProperty");

//create customData

CustomData data = new CustomData();

data.SetValue<string>(ConnectionStringProperty, "test");

string v = data.GetValue<string>(ConnectionStringProperty);
```

这个代码主要还是演示如何使用DebuggerDisplay和DebuggerTypeProxy功能。

详细的代码如下：

```csharp
// CustomProperty
//==========================================

//  File:           CustomProperty.cs

//  Date:           2008-03-10

//  Author:         tansm

//------------------------------

//

//  Description:

//  提供自定义的选项属性键

//

//  History:        2008-03-10    Created

//

//===========================================

using System;

namespace Dcms.Common.Advanced {

    /**//// <summary>

    /// 自定义选项属性的描述对象。被应用在ApplicationControlService的自定义属性描述

    /// </summary>

    /// <typeparam name="T">此属性的类型，必须是值类型　</typeparam>

    [

    Serializable,

    System.Diagnostics.DebuggerDisplay(@"{PropertyName}:{PropertyType}")

    ]

    public sealed class CustomProperty<T>  {

        /**//// <summary>

        /// 创建 CustomProperty 实例

        /// </summary>

        /// <param name="propertyName">属性的名称,不能为空</param>

        public CustomProperty(string propertyName) {

            参数检查#region 参数检查

            if (string.IsNullOrEmpty(propertyName)) {

                throw new ArgumentNullException("propertyName");

            }

            #endregion

            _propertyName = propertyName;

            _defaultValue = default(T);

            _isReadonly = false;

        }

        /**//// <summary>

        /// 创建 CustomProperty 实例

        /// </summary>

        /// <param name="propertyName">属性的名称,不能为空</param>

        /// <param name="defaultValue">属性的缺省值</param>

        /// <param name="isReadonly">此属性是否是只读</param>

        public CustomProperty(string propertyName, T defaultValue, bool isReadonly) {

            参数检查#region 参数检查

            if (string.IsNullOrEmpty(propertyName)) {

                throw new ArgumentNullException("propertyName");

            }

            #endregion

            _propertyName = propertyName;

            _defaultValue = defaultValue;

            _isReadonly = isReadonly;

        }

        private string _propertyName;

        /**//// <summary>

        /// 返回自定义属性的名称

        /// </summary>

        public string PropertyName {

            get { return _propertyName; }

        }

        /**//// <summary>

        /// 返回属性的类型

        /// </summary>

        public Type PropertyType {

            get { return typeof(T); }

        }

        private T _defaultValue;

        /**//// <summary>

        /// 返回此属性的默认值

        /// </summary>

        public T DefaultValue {

            get { return _defaultValue; }

        }

        private bool _isReadonly;

        /**//// <summary>

        /// 返回属性是否是只读属性

        /// </summary>

        public bool IsReadonly {

            get { return _isReadonly; }

        }

        /**//// <summary>

        /// 返回此对象的Hash值

        /// </summary>

        /// <returns>实例的Hash值</returns>

        public override int GetHashCode() {

            return System.Runtime.CompilerServices.RuntimeHelpers.GetHashCode(this);

        }

        /**//// <summary>

        /// 判断两个对象是否相等

        /// </summary>

        /// <param name="obj">要判断的实例</param>

        /// <returns>相等返回true</returns>

        public override bool Equals(object obj) {

            return object.ReferenceEquals(obj, this);

        }

        /**//// <summary>

        /// 返回此实例的字符串表示

        /// </summary>

        /// <returns>实例的字符串表示</returns>

        public override string ToString() {

            return string.Format("{0}:{1}", _propertyName, typeof(T));

        }

    }

}

CustomData

//==========================================

//  File:           CustomData.cs

//  Date:           2008-03-10

//  Author:         tansm

//------------------------------

//

//  Description:

//  自定义属性包装类

//

//  History:        2008-03-10    Created

//

//===========================================

using System;

using System.Collections.Generic;

using System.Diagnostics;

namespace Dcms.Common.Advanced {

    /**//// <summary>

    /// 包含自定义属性的类，此类被应用在ApplicationControlService

    /// </summary>

    [

    Serializable,

    System.Diagnostics.DebuggerDisplay(@"Count = {Count}"),

    DebuggerTypeProxy(typeof(CustomDataDebugView)),

    ]

    public class CustomData {

        private Dictionary<object, object> _values; //property为键

        /**//// <summary>

        /// 创建 CustomData 实例

        /// </summary>

        public CustomData() {

            _values = new Dictionary<object, object>();

        }

        /**//// <summary>

        /// 返回指定自定义属性的值，如果没有包含值，将返回默认值

        /// </summary>

        /// <typeparam name="T">属性的值类型</typeparam>

        /// <param name="property">属性键</param>

        /// <returns>此属性的值，如果没有找到将返回默认值</returns>

        public T GetValue<T>(CustomProperty<T> property) {

            参数检查#region 参数检查

            if (property == null) {

                throw new ArgumentNullException("property");

            }

            #endregion

            object result;

            if (_values.TryGetValue(property, out result)) {

                return (T)result;

            }

            return property.DefaultValue;

        }

        /**//// <summary>

        /// 设置某个属性的值，如果是只读属性只能设置一次。

        /// </summary>

        /// <typeparam name="T">属性的值类型</typeparam>

        /// <param name="property">属性键</param>

        /// <param name="value">属性的新值</param>

        public void SetValue<T>(CustomProperty<T> property, T value) {

            参数检查#region 参数检查

            if (property == null) {

                throw new ArgumentNullException("property");

            }

            #endregion

            if (_values.ContainsKey(property)) {

                if (property.IsReadonly) {

                    throw new ApplicationException(string.Format("自定义属性{0}是只读属性，只能设置一次。", property));

                }

                _values[property] = value;

            } else {

                _values.Add(property, value);

            }

        }

        /**//// <summary>

        /// 返回所有的属性列表

        /// </summary>

        /// <returns>属性列表 </returns>

        public object[] GetProperties() {

            object[] keys = new object[_values.Count];

            _values.Keys.CopyTo(keys, 0);

            return keys;

        }

        /**//// <summary>

        /// 返回属性的总数

        /// </summary>

        public int Count {

            get { return _values.Count; }

        }

        /**//// <summary>

        /// 将数据复制到数组中

        /// </summary>

        /// <param name="array">新的数组</param>

        /// <param name="arrayIndex">开始索引</param>

        internal void CopyTo(KeyValuePair<object, object>[] array, int arrayIndex) {

            ((ICollection<KeyValuePair<object, object>>)_values).CopyTo(array, arrayIndex);

        }

    }

    internal sealed class CustomDataDebugView {

        private CustomData _customData;

        public CustomDataDebugView(CustomData customData) {

            参数检查#region 参数检查

            if (customData == null) {

                throw new ArgumentNullException("customData");

            }

            #endregion

            this._customData = customData;

        }

        [DebuggerBrowsable(DebuggerBrowsableState.RootHidden)]

        public KeyValuePair<object, object>[] Items {

            get {

                KeyValuePair<object, object>[] values = new KeyValuePair<object, object>[_customData.Count];

                _customData.CopyTo(values, 0);

                return values;

            }

        }

    }

}
```
