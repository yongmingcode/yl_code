# 伊始
有这么一个场景：有一个抽奖活动，需要对几种奖品随机发放。我们可以将不同的奖品类别放入List，然后调用shuffle方法打乱这个List中元素的顺序，然后get(0),即可获得随机的一个奖品。
```java
    ArrayList<Integer> integerList = new ArrayList<>();
    integerList.add(0);
    integerList.add(1);
    integerList.add(2);
    Collections.shuffle(integerList);

    // 控制台可能打印0、1、2中的任何一个
    System.out.println(integerList.get(0));
```

## 方法作用：

> shuffle(List<?> list)：使用默认的随机源，随机排列指定的列表。所有排列将以近似相等的可能性发生。

> shuffle(List<?> list, Random rnd)：使用指定的随机源，随机排列指定的列表。假设随机性的来源是公平的，那么所有排列都会以相同的可能性发生。

# 方法原理

## 以下是jdk源码（1.8）：

```java
public static void shuffle(List<?> list, Random rnd) {
        int size = list.size();
        if (size < SHUFFLE_THRESHOLD || list instanceof RandomAccess) {
            for (int i=size; i>1; i--)
                swap(list, i-1, rnd.nextInt(i));
        } else {
            Object arr[] = list.toArray();

            // Shuffle array
            for (int i=size; i>1; i--)
                swap(arr, i-1, rnd.nextInt(i));

            // Dump array back into list
            // instead of using a raw type here, it's possible to capture
            // the wildcard but it will require a call to a supplementary
            // private method
            ListIterator it = list.listIterator();
            for (int i=0; i<arr.length; i++) {
                it.next();
                it.set(arr[i]);
            }
        }
    }

```
上面代码先判断list的size是否小于SHUFFLE_THRESHOLD(5)或者实现RandomAccess接口，如果符合条件，则执行swap(List<?> list, int i, int j)方法进行交换；如果不符合条件，则把list转为数组，数组执行swap(Object[] arr, int i, int j)（注意这里是另外一个swap()方法哦）方法进行交换，交换之后的数组循环转回List。

到这里我产生了几个疑问：
1. 那判断条件为什么是size是否小于SHUFFLE_THRESHOLD(5)和实现RandomAccess接口？
2. 为什么符合条件就直接交换？不符合条件就转为数组再交换？

这几个问题留在后面解答。

## 我们来看一下这两个swap()方法：


### swap(Object[] arr, int i, int j)

```java
private static void swap(Object[] arr, int i, int j) {
    Object tmp = arr[i];
    arr[i] = arr[j];
    arr[j] = tmp;
}
```
有三个参数：
- list：要交换元素的list。
- i、j: 是两个要交换元素的下标
  * i: shuffle()方法中是从list的最后一个下标开始
  * j：列表中随机的一个下标

然后让i、j下标的元素进行交换。


### swap(List<?> list, int i, int j)方法：
```java
public static void swap(List<?> list, int i, int j) {
    // instead of using a raw type here, it's possible to capture
    // the wildcard but it will require a call to a supplementary
    // private method
    final List l = list;
    l.set(i, l.set(j, l.get(i)));
}

```
i、j参数的含义和上面的swap(Object[] arr, int i, int j)方法含义相同，方法的功能也是简单的让i、j下标的元素进行交换。

> 但是swap(List<?> list, int i, int j)方法中有一个需要注意的地方：final List l = list，这里为什么不直接用list，而是通过使用副本 l 进行操作呢？使用中间变量（副本）是为了规避泛型的缺点!不理解可以参考文末的参考文档。




















---

参考资料：

1. [爲什麼Collections.swap複製輸入列表？](http://hk.uwenku.com/question/p-sakqoope-yt.html "爲什麼Collections.swap複製輸入列表？")
2. [为什么Collections.swap将目标列表分配给原始类型的变量？](https://www.coder.work/article/400392 "为什么Collections.swap将目标列表分配给原始类型的变量？")
3. [Java 泛型，你了解类型擦除吗？](https://blog.csdn.net/briblue/article/details/76736356 "Java 泛型，你了解类型擦除吗？")





