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

    // 这里考虑了一下多线程了情况
    // 如果当前集合还有未转入数组的元素，则调用finishToArray方法,进行扩容
    // 如果当前集合没有未转入数组的元素，则返回数组r
    return it.hasNext() ? finishToArray(r, it) : r;
}

```
上面新建了一个数组，将集合中的元素遍历放入数组。代码中对多线程环境下的情况进行了处理，如果有别的线程对集合进行了修改使得：集合元素少于预期，则截断返回;集合元素多于预期，则数组进行扩容添加，然后再返回数组。

# toArray(T[]):
```java
public <T> T[] toArray(T[] a) {

        // 1.预估集合大小
        int size = size();

        // 2. 为了准备一个合适的数组存储集合的元素：
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

        // 这里考虑了一下多线程了情况
        // 如果当前集合还有未转入数组的元素，则调用finishToArray方法,进行扩容
        // 如果当前集合没有未转入数组的元素，则返回数组r
        return it.hasNext() ? finishToArray(r, it) : r;
    }

```
