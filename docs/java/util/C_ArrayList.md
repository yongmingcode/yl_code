# 介绍
ArrayList类是List接口的实现类，就像它名字那样，“数组列表”即基于数组可调整大小的列表。允许存放null元素，与Vector相似，但去除了线程安全。其数据的顺序与插入的顺序一致。

JDK中这样定义该类：
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable

```
可以看到，ArrayList是[AbstractList](/java/util/c_AbstractList.md)的子类，同时实现了List接口。同时实现了三个标记接口：
- RandomAccess: 实现该接口的类，支持快速随机访问。
- Cloneable: 实现该接口的类，支持克隆（重写clone方法）。
- java.io.Serializable：实现该接口的类，支持序列化。


# ArrayList中的几个变量
在ArrayList中有几个成员变量我们需要注意：
```java
// 默认初始化容量
private static final int DEFAULT_CAPACITY = 10;

// 当实例大小为0时，ArrayList内部直接使用该数组
private static final Object[] EMPTY_ELEMENTDATA = {};

// 当向空实例中添加元素后，ArrayList内部直接使用该数组
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

// 数组缓冲区。当空实例添加完第一个元素，则实例的容量将扩展为DEFAULT_CAPACITY
// transient标明了该变量不用序列化，因为序列化时会直接读取该数组的元素进行序列化，所以就不用序列化该变量了，可参考ArrayList中的writeObject()方法
transient Object[] elementData;

// 为数组分配的最大容量
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
```

从这几个变量的定义中可以看出，ArrayList的实质就是数组。这就不难解释，为什么ArrayList能被标记为支持快速随机访问的类了（因为数组的访问，本身就是一种快速随机访问）。

# 构造方法
ArrayList提供了3个构造方法：
```java
// 初始化一个为指定size的list
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

// 初始化一个指定容量的list
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}

// 根据给定的Collection创建一个List（其实ArrayList就是一个Collection,不是吗？）
public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
    if ((size = elementData.length) != 0) {
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // replace with empty array.
        this.elementData = EMPTY_ELEMENTDATA;
    }
}

```

# 重要方法

## ArrayList中与[AbstractList](/java/util/c_AbstractList.md)区别的一些新实现
以下方法，在[AbstractList](/java/util/c_AbstractList.md)中都是基于Iterator实现的，而在ArrayList中却不是:
- indexOf(Object o)：不再通过Iterator来实现
- lastIndexOf(Object o)：不再通过Iterator来实现
- toArray()：不再通过Iterator来实现，可能会截断原集合
- toArray(T[] a)：带类型的转换。转数组时，尽量使用该方法。

我们应该这样使用:
```java
List<String> list = new ArrayList<>(2);
list.add("guan");
list.add("bao");
String[] array = new String[list.size()];
array = list.toArray(array);
```

## set()方法
将指定元素替换到指定位置，并返回原位置上的元素。
```java
public E set(int index, E element) {
    rangeCheck(index);

    E oldValue = elementData(index);
    elementData[index] = element;
    return oldValue;
}
```
## 添加
添加涉及到ArrayList的内存扩容问题，所以更加值得关注。
```java
// 1.在列表末尾添加元素
public boolean add(E e) {
    // 确保数组容量足够
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}

// 2.确保数组容量足够的方法
private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}

// 3.计算容量
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    // 如果是第一个添加的元素，则容量为DEFAULT_CAPACITY
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}

// 5.又执行了ensureExplicitCapacity方法进行容量判断
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // 超出容量则扩容
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

// 5.扩容
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;

    // 扩容为原来的1.5倍
    // 新容量 = 旧容量*1/2 + 旧容量
    int newCapacity = oldCapacity + (oldCapacity >> 1);

    // 扩容后的还不够，则直接用给定的容量minCapacity
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;

    // 这里对最终的容量判断内存是否溢出
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}

// 6.最大容量限度处理及内存溢出判定
private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}
```
OK，到这里内存扩容机制就清楚了。首次创建一个空数组，第一次向集合中插入数据的时候，空数组容量扩容为10。如果容量不够，则扩充为原来的1.5倍，如果还不够，就需要给定的长度作为集合的新容量了。

一个问题：
> 上面的扩容机制，一定程度上缓解了扩容时频繁copy的问题，但当处理大量数据时，还是会出现频繁copy的问题，ArrayList为我们提供了两种可行的方案：
> 1. 使用带有初始化容量的构造器ArrayList(int initialCapacity)。
> 2. 预测到添加的数据量可能比较大，那么先进行扩容，再添加数据。扩容方法ensureCapacity(int minCapacity)，该方法调用的还是ensureExplicitCapacity(int minCapacity)

# 其它部分方法
还有一些方法，这里只列举出方法作用，不做详细说明了：
- trimToSize()：将elementData的大小设置为size大小，释放无用内存
- remove(int index)：删除指定位置的元素，并返回该元素。
- remove(Object o)：删除在集合中第一次出现的该元素。
- removeRange(int fromIndex, int toIndex)：删除指定索引范围的元素
- writeObject(java.io.ObjectOutputStream s): 序列化

# 总结
经过上面的分析，我们知道ArrayList就是基于数组实现的，所以方便随机访问，但不适合大量数据的插入。如果要进行数据的插入，可以考虑一下三种方式：
1. 使用带有初始化容量的构造器ArrayList(int initialCapacity)。
2. 使用ensureCapacity(int minCapacity)，先扩容再插入。
3. 使用LinkedList。
