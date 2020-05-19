# 介绍——jdk 1.8 toArray方法分析
在AbstractCollection中重载了2个toArray方法，都是为了将集合转数组。功能上来看，二者最大的区别就是返回的数组类型不同，一个是返回Object[],另一个返回的是指定的类型。但二者的实现过程差别比较大。

# toArray():
## 源码分析（jdk 1.8）：
```java
public Object[] toArray() {
    // 准备了一个和集合size()大小的数组
    Object[] r = new Object[size()];
    Iterator<E> it = iterator();
    for (int i = 0; i < r.length; i++) {
        if (! it.hasNext()) // 元素少于预期
            return Arrays.copyOf(r, i); // 截取数组r的前i个元素并返回。
        r[i] = it.next();
    }

    // 这里考虑了一下多线程情况
    // 如果当前集合还有未转入数组的元素，则调用finishToArray方法,进行扩容
    // 如果当前集合没有未转入数组的元素，则返回数组r
    return it.hasNext() ? finishToArray(r, it) : r;
}

```
上面新建了一个数组，将集合中的元素遍历放入数组。代码中对多线程环境下的情况进行了处理，如果有别的线程对集合进行了修改使得：集合元素少于预期，则截断返回;集合元素多于预期，则数组进行扩容添加，然后再返回数组。

如果数组的长度不够，这里调用了finishToArray方法进行扩容：
```java
private static <T> T[] finishToArray(T[] r, Iterator<?> it) {
    int i = r.length;

    // 1.从集合中多出来的元素（即数组装不下的元素）开始迭代
    while (it.hasNext()) {
        int cap = r.length; // 每次重新获取新数组r中元素的数量，供下面进行对比，保证数组r有容量储存迭代器it中的元素
        if (i == cap) { // 2.下面是扩容操作。如果

            // 2.1 进行扩容：容量变为原容量的1.5倍，capp + cap/2 + 1
            int newCap = cap + (cap >> 1) + 1;

            // 2.2 判断新容量是否溢出，即超出最大容量
            if (newCap - MAX_ARRAY_SIZE > 0)
                newCap = hugeCapacity(cap + 1);
            // 2.3 将数组r中的元素放入一个长度为newCap的新数组中
            r = Arrays.copyOf(r, newCap);
        }
        r[i++] = (T)it.next();
    }
    // trim if overallocated
    return (i == r.length) ? r : Arrays.copyOf(r, i);
}

```
finishToArray方法对数组进行扩容，在数组不会溢出的前提下，即数组长度没有达到最大长度的前提下，保证数组r中有足够的空间存储迭代器it中的元素（采取容量不够就扩容的方式）。

最大容量判断：
```java
private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // 如果一个int值加1变为负值，表示该int值已经超出最大长度，内存溢出
        throw new OutOfMemoryError
            ("Required array size too large");

    // 如果最小容量大于MAX_ARRAY_SIZE，则返回java中Integer的最大长度（说白了就是连MAX_ARRAY_SIZE保留的8位头字段也不要了），
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}

```

这里判断了数组溢出的情况，并对溢出的数组给定了最大的长度（如果最大长度也没法满足，那就没办法了）。

> int值加1为什么会变成负数？
> 计算机存储的数值，都是用补码进行表示。即首位都是符号位，0表示正数，1表示负数。如int是32位，则第一位是符号位，其余的31位进行计算（二进制位运算）。所以最大的int值是：

> 0111 1111 1111 1111 1111 1111 1111 1111

> 加1：

> 1000 0000 0000 0000 0000 0000 0000 0000

> 符号位是1，所以是负值。

# toArray(T[]):
```java
public <T> T[] toArray(T[] a) {

        // 1.预估集合大小
        int size = size();

        // 2. 准备一个合适的数组存储集合的元素：
        // 2.1 如果a的长度大于集合的长度，则直接向a中存元素就行，即让r和a指向同一个对象
        // 2.2 否则，创建一个与a类型相同且大小相等的数组
        T[] r = a.length >= size ? a :
                  (T[])java.lang.reflect.Array
                  .newInstance(a.getClass().getComponentType(), size);
        Iterator<E> it = iterator();

        for (int i = 0; i < r.length; i++) {
            if (! it.hasNext()) { // 元素少于预期
                if (a == r) { // 如果a和r指向同一个对象
                    r[i] = null; // 把r上第i位，置为null
                } else if (a.length < i) { // 这里的判断表明，r其实是指向第2.2步中新建的数组。同时也表明，集合中的元素a数组装不下。所以直接截断数组r，并返回。
                    return Arrays.copyOf(r, i); // 截断数组r。
                } else {
                // a数组装能装下集合中的元素，那么就将r数组中的元素装入a数组。
                    System.arraycopy(r, 0, a, 0, i);

                    if (a.length > i) { //a装完集合中的元素还有空余空间
                        a[i] = null; // 则将剩余的空间置为null
                    }
                }
                return a;
            }
            r[i] = (T)it.next();
        }

        // 这里考虑了一下多线程情况
        // 如果当前集合还有未转入数组的元素，则调用finishToArray方法,进行扩容
        // 如果当前集合没有未转入数组的元素，则返回数组r
        return it.hasNext() ? finishToArray(r, it) : r;
    }

```
该方法传入一个指定类型的数组，并返回一个指定类型的数组。

通过上面的代码分析不难看出，使用该方法，最好是传入一个与集合长度相同的数组，不然会产生资源浪费：
1. 传入的数组空间不够，则toArray方法内部会创建一个新的数组，重新分配内存空间；
2. 传入的数组空间过大，多出来的空间被置为null，造成资源浪费。

所以我们可以这样使用:
```java
List<String> list = new ArrayList<>(2);
list.add("guan");
list.add("bao");
String[] array = new String[list.size()];
array = list.toArray(array);
```

# 总结
通过上面的分析，对于集合转数组，我们得出以下结论：
1. 无参的toArray方法，只能返回Object[]类型的数组，其实我们需要Object[]类型的数组的情况还是比较少的，所以建议使用有参的toArray方法。
2. 有参的toArray方法使用时，传入的数组类型应该和集合元素的类型相同，且数组的大小为集合大小的数组。
