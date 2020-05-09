# 概述
copyOf方法是Arrays类里经常被使用到的一个方法，从名字中我们可以看出它主要功能就是实现数组之间的复制，Arrays类中重载了10个同名方法来应对不同数据类型的数组复制，那SE开发者是如何设计的呢？下面我们就来一起看一下这个方法吧。

# 方法详解

## copyOf(T[], int)

方法源码：
```java
// original-要复制的数组，newLength-要返回的副本的长度
public static <T> T[] copyOf(T[] original, int newLength) {
    return (T[]) copyOf(original, newLength, original.getClass());
}
```
- 参数说明：
    - original：要复制的数组，
    - newLength：要返回的副本的长度

这里调用了另外一个重载的copyOf方法：
```java
public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
    @SuppressWarnings("unchecked")
    T[] copy = ((Object)newType == (Object)Object[].class)
        ? (T[]) new Object[newLength]
        : (T[]) Array.newInstance(newType.getComponentType(), newLength);
    System.arraycopy(original, 0, copy, 0,
                     Math.min(original.length, newLength));
    return copy;
}
```
- 参数说明：
    - original：要复制的数组，
    - newLength：要返回的副本的长度
    - newType：要返回副本的类型

一行行来看：

1. (Object)newType == (Object)Object[].class：判断newType是不是Object类型数组。因为用到了==号，比较的是内存地址，所以需要进行向上强转为Object才能比较，不然编译为抛异常说不可比较的类型(java中同一类型的对象才能进行比较)。
2. 在三目运算符中，如果newType是Object类型，则新创建一个数组对象；不然则执行Array.newInstance()方法。
3.
