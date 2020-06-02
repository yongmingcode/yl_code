# AbstractList
AbstractList类是List接口的超级实现类，里面实现了大部分的List接口中的方法。AbstractList类同时也是AbstractCollection抽象类的子类，对AbstractCollection中没有实现的iterator()方法进行了具体实现。所以AbstractList类同时具备List接口和AbstractCollection抽象类的特点。

> 如果要实现一个不可修改的集合，只需要复写get和size就可以了；要实现一个可以修改的集合，还需要复写set方法；如果要动态调整大小，就必须再实现add和remove方法。

Java中有序集合都直接或间接的继承该类，这就意味着到AbstractList这一层，，有序集合的基本方法都会被封装，这些方法具有普适性，可被Stack、ArrayList、LinkedList使用。

下面就让我们一起看一下AbstractList对集合的操作提供了哪些方法。

# 源码分析

## 两个内部类
在AbstractList类中定义了两个内部类Itr和ListItr，它们的定义如下：
``` java
// Itr类提供了对Iterator类中方法的实现
private class Itr implements Iterator<E>

// ListItr提供了对ListIterator方法的实现
private class ListItr extends Itr implements ListIterator<E>
```

## 变量
有一个值得注意的变量：
```java
// 定义在AbstractList类中，该变量用于记录AbstractList子类在实现iterator、listIterator时使用，主要作用是记录修改次数。
// 即每次使用iterator、listIterator时，modCount的值都会赋予expectedModCount，每次操作前expectedModCount与modCount对比，不相等则表示可能有其它情况（如其它线程）对实例进行了修改，会抛出ConcurrentModificationException异常
// 这就是fail-fast机制
protected transient int modCount = 0;

// 定义在AbstractList的内部类Itr中
int expectedModCount = modCount;
```
举个该变量使用的例子，以下是next()方法源码：
```java
public E next() {
    // 通过modCount检查实例是否有变动
    checkForComodification();
    try {
        int i = cursor;
        E next = get(i);
        lastRet = i;
        cursor = i + 1;
        return next;
    } catch (IndexOutOfBoundsException e) {
        checkForComodification();
        throw new NoSuchElementException();
    }
}

final void checkForComodification() {
    // 二者不同则表示实例被修改，抛出异常
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
}
```


## add、indexOf、lastIndexOf、clear、removeRange方法分析
AbstractList类中提供了add方法的实现，直接将元素添加到集合的末尾，而addAll方法则是通过调用迭代器，从指定位置开始，逐个将元素添加到集合中，下面是add方法的源码：
```java
public boolean add(E e) {
    add(size(), e);
    return true;
}
```

在该类中，indexOf、lastIndexOf、clear、removeRange方法都用到了Iterator或是ListIterator:
```java
public Iterator<E> iterator() {
    return new Itr();
}

public ListIterator<E> listIterator(final int index) {
    rangeCheckForAdd(index);

    return new ListItr(index);
}
```
是的，在AbstractList中，Iterator和ListIterator都是内部类并实现了Iterator接口，很自然的可以使用hasNext、next、remove方法对集合元素进行操作了。

## subList方法及SubList类分析
值得注意的是，subList方法是操作的原集合，返回指定范围的原集合，即允许用户操作原集合的部分范围。

SubList类是AbstractList的子类，subList方法的功能就是在SubList类中实现的。

## equals和hashcode方法
这两个方法分别对集合和集合中的元素进行了判断，所以如果想正确的使用这两个方法，则需要对这两个方法进行复写。

# 总结
AbstractList类主要是对集合接口的及其父类的方法进一步实现，该类中的方法对有序集合的操作更具有针对性。java通过这种层级间的方法实现，实现对各个类的高度利用。

