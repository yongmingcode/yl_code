# ArrayList遍历删除元素
回想起来，开发过程中遇到过很多坑，一步步的走来，确实真的挺不容易。ArrayList遍历删除就有一个坑，如果不从源码上分析，很难了解这个坑产生的原因。我阅读源码的习惯就是从这个坑开始的，所以要好好记录一下。

# 问题复现
在List中定义元素：Monday、Tuesday、Thresday，遍历去掉List中元素为"Monday"的元素，打印输出其它元素。
## for循环删除元素
### 问题分析
easy，代码实现如下：
```java
List<String> list = new ArrayList<>(3);
list.add("Monday");
list.add("Tuesday");
list.add("Thresday");
for(int i = 0; i < list.size(); i++){
    if("Monday".equals(list.get(i))){
        list.remove(list.get(i)); // position_1
        continue;
    }
    System.out.println(list.get(i));
}
```
具体实现是使用for循环遍历了一个List,当List中的元素为"Monday"时，调用ArrayList的remove方法删除"Monday"这个元素。

控制台输出：thresday

纳尼？我要的“Thresday”呢？来掰着手指头分析一下：

第一次循环，循环变量i（即下标）的值为0，List中是这样的：

| 0      | 1       | 2        |
| --     | --      | --       |
| Monday | Tuesday | Thresday |

执行完“position_1”处的代码后变成：

| 0       | 1        |
| --      | --       |
| Tuesday | Thresday |

由于continue的存在，第一次循环执行完不会打印任何东西。

第二次循环，而此时循环变量i值为1，所以直接打印了“Thresday”，完美的忽略了“Tuesday”。

### 解决方法
既然知道了问题，那本着方法总比问题多的原则，试着改进一下。通过上面的分析可以知道是由于删除完元素，循环变量i的值变大了，那么很好办：
```java
List<String> list = new ArrayList<>(3);
list.add("monday");
list.add("tuesday");
list.add("thresday");
for(int i = 0; i < list.size(); i++){
    if("monday".equals(list.get(i))){
        list.remove(list.get(i));
        i--; // 增加i--修复删除完元素i值变大的问题。
        continue;
    }
    System.out.println(list.get(i));
}
```

一个小问题：上面的代码能优化一下吗？当然可以，请接着往下看。

### 倒序for循环删除元素
倒叙遍历方式实现：
```java
List<String> list = new ArrayList<>(3);
list.add("monday");
list.add("tuesday");
list.add("thresday");
for(int i = list.size() - 1; i >= 0; i--){
    if("thresday".equals(list.get(i))){
        list.remove(list.get(i)); // position_2
        continue;
    }
    System.out.println(list.get(i));
}
```
这种方式的实现解决了每次删除完要修正下标（即循环变量i）的问题,可以来分析一下为什么：

第一次循环，i的值为2，List中是这样的：

| 0      | 1       | 2        |
| --     | --      | --       |
| Monday | Tuesday | Thresday |

执行完“position_2”处的删除操作后，continue;所以第一次循环什么都不输出。

第二次循环，i的值为1，List中是这样的：

| 0      | 1       |
| --     | --      |
| Monday | Tuesday |

所以刚好取得我想要的值“Tuesday”。这是巧合吗？不是的。删除一次元素，每次下面都减1，所以这是逻辑上的正确表达。

## for-each循环删除元素
程序的世界就是那么让人兴奋。后来,我又认识了“A new girl”——for-each，她可以提供更加方便的遍历操作。直接上......代码：
```java
List<String> list = new ArrayList<>(3);
list.add("monday");
list.add("tuesday");
list.add("thresday");
for(String str : list){
    if("monday".equals(str)){
        list.remove(str);
        continue;
    }
    System.out.println(str);
}
```
\- -！比上次还严重，程序直接抛异常：java.util.ConcurrentModificationException。I did not do anything......

### 问题分析
为什么会抛出ConcurrentModificationException异常呢？拿出我的尚方宝剑：javap，以下是反编译结果：
```java
37: invokeinterface #8,  1            // 调用方法： java/util/List.iterator()方法
42: astore_2
43: aload_2
44: invokeinterface #9,  1            // 调用方法： java/util/Iterator.hasNext()方法
49: ifeq          92
52: aload_2
53: invokeinterface #10,  1           // InterfaceMethod java/util/Iterator.next()方法
```
从上面可以看出，for-each循环是通过调用Iterator迭代器实现的，而Iterator的核心方法是hasNext和next方法。而在ArrayList类中的内部类Itr，就是对Iterator接口的实现类。
以下列出Itr的部分源码：
```java
private class Itr implements Iterator<E> {
    int cursor;       // index of next element to return
    int lastRet = -1; // index of last element returned; -1 if no such
    int expectedModCount = modCount;

    Itr() {}

    public boolean hasNext() {
        return cursor != size;
    }

    @SuppressWarnings("unchecked")
    public E next() {
        checkForComodification(); // position_3
        int i = cursor;
        if (i >= size)
            throw new NoSuchElementException();
        Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length)
            throw new ConcurrentModificationException();
        cursor = i + 1;
        return (E) elementData[lastRet = i];
    }

    final void checkForComodification() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }
}

```
我们发现在next方法的position_3处调用的checkForComodification方法中，当modCount和expectedModCount不相等时，会抛出ConcurrentModificationException异常。在[AbstractList](java/util/c_AbstractList)中我们分析过,modCount时记录集合被修改的次数。由此可知，抛出异常时modCount和expectedModCount不相等。通过跟踪源码我发现，在执行如下代码时:
```java
list.remove(str);
```
modCount值被加1：
```java
public boolean remove(Object o) {
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}

private void fastRemove(int index) {
    modCount++; // 这里modCount的值被加1
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work
}
```
所以在循环再次执行到next()方法时，modCount和expectedModCount不相等。

### 解决方法
既然不能使用ArrayList中的remove方法，我们该如何实现删除呢？

在内部类Itr中，有一个remove方法,所以我们直接将代码改成：
```java
List<String> list = new ArrayList<>(3);
list.add("monday");
list.add("tuesday");
list.add("thresday");
Iterator<String> iterator = list.iterator();
while (iterator.hasNext()){
    String str = iterator.next();
    if("monday".equals(str)){
        iterator.remove();
        continue;
    }
    System.out.println(str);
}
```
控制台输出：
```java
tuesday
thresday
```
让我们看一下Itr中remove方法的源码：
```java
public void remove() {
    if (lastRet < 0)
        throw new IllegalStateException();
    checkForComodification();

    try {
        ArrayList.this.remove(lastRet);
        cursor = lastRet;
        lastRet = -1;
        expectedModCount = modCount; // 删除完，使两个变量相等
    } catch (IndexOutOfBoundsException ex) {
        throw new ConcurrentModificationException();
    }
}
```
从源码可看出，在删除完元素后，会将modCount重新赋值给expectedModCount。

## removeIf()方法循环遍历删除元素
```java
List<String> list = new ArrayList<>(3);
list.add("monday");
list.add("tuesday");
list.add("thresday");
System.out.println(list);
list.removeIf(str -> "tuesday".equals(str));
System.out.println(list);
```

# 拓展延伸
在使用for-each循环时还会遇到一个问题，就是在使用ArrayList的remove方法删除元素后，没有抛出ConcurrentModificationException异常：
```java
List<String> list = new ArrayList<>(3);
list.add("monday");
list.add("tuesday");
list.add("thresday");
for(String str : list){
    if("tuesday".equals(str)){ // 注意这里删除的时tuesday
        list.remove(str);
        continue;
    }
    System.out.println(str);
}
```
但是控制台只输出了：monday

上面的分析我们已经知道，for-each循环其实时通过Iterator实现的，所以我们直接把上面代码转成Iterator实现的方式：
```java
List<String> list = new ArrayList<>(3);
list.add("monday");
list.add("tuesday");
list.add("thresday");
Iterator<String> iterator = list.iterator();
while (iterator.hasNext()){
    String str = iterator.next();
    if("tuesday".equals(str)){
        list.remove();
        continue;
    }
    System.out.println(str);
}
```
控制台还是只输出了：monday

其实这个还是涉及到Itr内部类的源码，其中hasNext()方法源码如下：
```java
int cursor;       // 光标，返回下一个元素的索引
public boolean hasNext() {
    return cursor != size;
}
```
可以知道，while循环执行循环体的条件时光标不等于集合大小，即集合循环的下一个元素存在。我们调用完下面的代码后:
```java
list.remove(str);
```
集合中的元素会减少一个，所以集合的size就减一。

再回顾以下next()方法源码：
```java
public E next() {
    checkForComodification(); // position_3
    int i = cursor;
    if (i >= size)
        throw new NoSuchElementException();
    Object[] elementData = ArrayList.this.elementData;
    if (i >= elementData.length)
        throw new ConcurrentModificationException();
    cursor = i + 1; // 光标加1
    return (E) elementData[lastRet = i];
}
```
每次调用完next方法，光标都会加1。到这里我们就可以很确定ArrayList中Itr循环过程了：每次调用hasNext通过光标的值和集合的size来判断集合中是否还有元素需要循环，每次调用next方法，都会对光标增加1，知道光标的值与集合的size相等，则表示集合中的元素已经被遍历完成。

回到问题：

第二次循环，cursor光标的值为1，集合size为2。

第二次循环删除元素，光标值没有变化还是1，集合size减小1变为1。

第三次循环调用hasNext发现cursor == size，循环结束，自然就不会再打印thresday了。

# 总结
循环遍历删除元素，有四种有效的方法：
- 使用removeIf()方法
- 使用Iterator的remove()方法
- 使用for循环正序遍历
- 使用for循环倒序遍历

看似一个简单的循环遍历删除，如果不了解源码，很有可能掉坑里。加油~！奥里给！！！
