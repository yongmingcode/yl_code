# 概述
copyOf方法是Arrays类里经常被使用到的一个方法，该方法返回一个新的数组，这个新数组有什么特点呢？下面我们就来一起看一下这个方法吧。

> 特别说明：文中引入了其它文章，只是提供深入了解copyOf方法源码中涉及其它方法的途径，即便不去阅读这些引用，笔者相信也是可以读懂copyOf方法的。

# 方法详解

## copyOf(T[], int)

### 方法源码：
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

### 分析：

1. (Object)newType == (Object)Object[].class：判断newType是不是Object类型数组。（因为用到了==号,java中同一类型的对象才能进行比较，比较的是内存地址，所以需要进行向上强转为Object才能比较，不然编译会抛异常：不可比较的类型。）。
    1. 如果newType是Object类型，则新创建一个长度为newLength的Object类型的数组。
    2. 如果newType不是Object类型，则调用[Array.newInstance()](/java/lang/reflect/C_Array.md)方法。该方法创建了一个类型为newType的元素类型、长度为newLength的数组。
    > [getComponentType()](/java/lang/C_Class.md): 该方法返回newType数组中的元素类型。

2. 然后执行了[System类](/java/lang/C_System.md)的静态方法[arraycopy](/java/lang/system/M_arraycopy.md)进行数组间的复制。（由Math.min(original.length, newLength)可看出，如果输入的长度newLength小于original数组的长度，那么该arraycopy方法会截取original数组，返回一个[0,newLength]的新数组）。

### 总结：
通过上面的分析，我们可以总结出copyOf(T[], int)方法的作用:
1. 当给定长度大于原来数组时，返回一个包含原来数组元素的新数组。
2. 当给定长度小于给定数组时，截断数组，方法会返回一个截断后的数组。

> 需要注意，这里调用[System](/java/lang/C_System.md)类的[arraycopy](/java/lang/system/M_arraycopy.md)静态方法，所以涉及到深复制和浅复制的问题。

### 一个小问题：

在(T[]) new Object[newLength]中，为什么Object[]可以强制转换成T[]呢？
> 解答：判断(Object)newType == (Object)Object[].class
1. 为true时，执行(T[]) new Object[newLength]，那T就是Object，newType也是Object[].class，所以可以强转成T[]
2. 为false时，执行的是(T[]) Array.newInstance(newType.getComponentType(), newLength)，Array.newInstance返回的本质就是T[]，所以可以强转成T[]。

## 重载的copayOf方法

有了上面对copyOf(T[], int)方法的分析，其余重载的copayOf方法都很简单了，这里只
以byte[]类型的copayOf方法为例进行讲解，其余的方法都相似：
1.copayOf(byte[], int):
```java
public static byte[] copyOf(byte[] original, int newLength) {
    byte[] copy = new byte[newLength];
    System.arraycopy(original, 0, copy, 0,
                     Math.min(original.length, newLength));
    return copy;
}
```
在该方法中，因为已经确定了original的类型，所以就不用对其类型进行判断并向上转型了。

作用：
1. 截断数组，当给定长度小于给定数组时，就可以实现截断的效果

其余的还有short、int、long、float、double、boolean、char类型对应数组的copayOf方法，作用都相同，就不再赘述了。

---

参考文档：

1. [Arrays.copyOf()方法详解-jdk1.8](https://my.oschina.net/u/2935389/blog/3030649 "Arrays.copyOf()方法详解-jdk1.8")
