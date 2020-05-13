# c_AbstractCollection
AbstractCollection是唯一一个实现Collection接口的类，并且是一个抽象类。我们可以先看一下它的方法实现，再去分析这个类的作用。

# 方法
主要列出AbstractCollection重写父类的一些方法及其独有的方法。

1. 构造方法
```java
protected AbstractCollection() {}
```

2. 属性相关方法：
  - isEmpty()：判断集合是否为空，其标准是集合size()是否等于0。
  - contains(Object o)：判断集合中是否包含某个元素。
    ```java
        public boolean contains(Object o) {

            // 通过迭代器迭代进行比较集合中每个元素是否相等
            Iterator<E> it = iterator();
            if (o==null) {// 分为null和不是null进行处理
                while (it.hasNext())
                    if (it.next()==null)
                        return true;
            } else {
                while (it.hasNext())
                    if (o.equals(it.next()))
                        return true;
            }
            return false;
        }
    ```
  - [toArray()](/java/util/abstractcollection/M_toArray.md): 将集合转为Object类型的数组。
  - [toArray(T[])](/java/util/abstractcollection/M_toArray.md)：将集合转为指定类型的数组。
    ```java
        public Object[] toArray() {
            // 准备了一个和集合size()大小的数组
            Object[] r = new Object[size()];
            Iterator<E> it = iterator();
            for (int i = 0; i < r.length; i++) {
                if (! it.hasNext()) // 元素少于预期
                    return Arrays.copyOf(r, i); // 截断数组r。
                r[i] = it.next();
            }
            return it.hasNext() ? finishToArray(r, it) : r;
        }
    ```
