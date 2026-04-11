# 对比Java的ArrayList与.net的List<T>

> 原文：https://www.cnblogs.com/tansm/archive/2012/12/20/2826217.html

今天看见一位园友写了一篇非常详细的文章《[ArrayList源码分析](http://www.cnblogs.com/hzmark/archive/2012/12/20/ArrayList.html)》，很佩服写的如此仔细和详细。

在看这篇文章时，我也和.net的List<T>做了对比，因为我非常熟悉List<T>的实现，就写了此篇文章说明各自实现的差异。

- 存储

```text
//Java
private transient Object[] elementData;
```

Java的此类虽然对外是泛型的，但内部却不是使用泛型的数组存储，没有.net好；

```text
//.net
private T[] _items;
```

- 默认构造

```text
//Java
public ArrayList() {
this(10);
```

Java默认构建了大小为10的数组，事实上，很多的时候我们开始根本不存储数据，而.net默认是0，而且使用一个静态的0数组实例，.net要聪明的多。

```text
//.net
public List()
{
    this._items = List<T>._emptyArray;
}
```

- 添加数据

```text
//java
public boolean add(E e) {
    ensureCapacity(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
    }
```

显然，无论数组是否大小够用，java都会尝试调用ensureCapacity方法，虽然此方法大多数并不会扩大数组（足够空间），但这造成了多调用一次方法。.net在这一块非常的小心。

```text
//.net
public void Add(T item)
{
    if (this._size == this._items.Length)
    {
        this.EnsureCapacity(this._size + 1);
    }
    this._items[this._size++] = item;
    this._version++;
}
```

在扩充多大的问题上，大家的意见似乎不一致，

```text
//Java
     public void ensureCapacity(int minCapacity) {
     modCount++;
     int oldCapacity = elementData.length;
     if (minCapacity > oldCapacity) {
         Object oldData[] = elementData;
         int newCapacity = (oldCapacity * 3)/2 + 1;
             if (newCapacity < minCapacity)
         newCapacity = minCapacity;
             // minCapacity is usually close to size, so this is a win:
             elementData = Arrays.copyOf(elementData, newCapacity);
     }
     }
```

Java使用奇怪的比例，增加2/3大小，而.net似乎很简单，增加2倍，具体谁更科学，我还做不出判断。

```text
//.net
private void EnsureCapacity(int min)
{
    if (this._items.Length < min)
    {
        int num = (this._items.Length == 0) ? 4 : (this._items.Length * 2);
        if (num > 2146435071)
        {
            num = 2146435071;
        }
        if (num < min)
        {
            num = min;
        }
        this.Capacity = num;
    }
}
```

- 添加一批数据

```text
//java
public boolean addAll(int index, Collection<? extends E> c) {
     if (index > size || index < 0)
         throw new IndexOutOfBoundsException(
         "Index: " + index + ", Size: " + size);

     Object[] a = c.toArray();
     int numNew = a.length;
     ensureCapacity(size + numNew);  // Increments modCount

     int numMoved = size - index;
     if (numMoved > 0)
         System.arraycopy(elementData, index, elementData, index + numNew,
                  numMoved);

         System.arraycopy(a, 0, elementData, index, numNew);
     size += numNew;
     return numNew != 0;
     }
```

 

添加一批数据时，在扩充大小时，java首先做一次复制(toArray）到临时数组，然后再复制到实际数组elementData。

```text
//.net
ICollection<T> collection2 = collection as ICollection<T>;
    if (collection2 != null)
    {
        int count = collection2.Count;
        if (count > 0)
        {
            this.EnsureCapacity(this._size + count);
            if (index < this._size)
            {
                Array.Copy(this._items, index, this._items, index + count, this._size - index);
            }
            if (this == collection2)
            {
                Array.Copy(this._items, 0, this._items, index, index);
                Array.Copy(this._items, index + count, this._items, index * 2, this._size - index);
            }
            else
            {
                T[] array = new T[count];
                collection2.CopyTo(array, 0);
                array.CopyTo(this._items, index);
            }
            this._size += count;
        }
    }
```

.net 的代码我看起来非常的奇怪，理论上，他可以调用collection2的copyto直接存入this._items的指定位置，这样就减少了一次复制，但不知道为什么他没有这样做。

- 其他的不同

判断项目相等，java用的是对象的相等方法，而.net用的是泛型版本的相等运算，效率更高。Clear时清除数据内容时使用for循环，而.net调用了Array的快速方法。

还有其他的不同嘛？我目前没有找到，感觉大部分的时候差别不大，可能.net稍微细致一些，你觉得呢？
