# 介绍
arraycopy作用主要是复制数组，从其名字不是驼峰写法就可看出其历史之久远。数组的复制涉及到深复制和浅复制的问题，我认为比较重要，所以这里摘出来单独进行讲解。

# 方法定义
```java
public static native void arraycopy(Object src,  int  srcPos,
                                        Object dest, int destPos,
                                        int length);
```
- 参数：
    - src：源数组
    - srcPos：源数组中读取数据的起始位置
    - dest: 目标数组
    - destPos: 写入目标数组的其实位置
    - length: 要复制的长度

该方法为native方法，具体由java虚拟机实现。

# 方法作用：
从数组src中复制元素数组desc。也就是说，把原数组src[srcPos,srcPos+length-1]位置的元素，复制到dest的[destPos,destPos+length-1]位置上。

这是一种覆盖性的复制，即目标数组dest复制之前destPos,destPos+length-1位置上的数据会被覆盖。

# 深复制与浅复制
1. 当数组为一维数组，且元素为基本类型或String类型时，属于深复制，即原数组与新数组的元素不会相互影响。
2. 数组为多维数组，或一维数组中的元素为引用类型时，属于浅复制，原数组与新数组的元素引用指向同一个对象


---

参考资料：

1. [System.arraycopy()方法详解](https://blog.csdn.net/balsamspear/article/details/85069207 "System.arraycopy()方法详解")

