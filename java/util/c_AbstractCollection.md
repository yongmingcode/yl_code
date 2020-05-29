# c_AbstractCollection
AbstractCollection是唯一一个实现Collection接口的类，并且是一个抽象类。我们可以先看一下它的方法实现，再去分析这个类的作用。

# 源码分析

### 常量
1. MAX_ARRAY_SIZE：要分配的最大数组大小。jdk中定义如下：
```java
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
```


### 构造方法
```java
protected AbstractCollection() {}
```

### 相关方法：
1. 属性操作
  - isEmpty()：判断集合是否为空，其标准是集合size()是否等于0。
  - [toArray()](/java/util/abstractcollection/M_toArray.md): 将集合转为Object类型的数组。
  - [toArray(T[])](/java/util/abstractcollection/M_toArray.md)：将集合转为指定类型的数组。
  - [finishToArray(T[], Iterator<?>)](/java/util/abstractcollection/M_toArray.md)：对集合进行扩容。
  - [hugeCapacity(int)](/java/util/abstractcollection/M_toArray.md)：溢出处理，即最大容量判断处理。
  - toString()：重写的toString方法。将集合中的元素以[a,b]的形式，使用StringBuilder拼接成字符串。

2. 添加
  - addAll(Collection<? extends E>): 通过for-each(其实还是迭代器)遍历每一个元素，通过add方法将元素添加到集合中。

3. 删除
  - remove(Object): 通过迭代器进行遍历寻找要删除的元素，并通过迭代器的remove方法删除元素。
  - removeAll(Collection<?>): 集合求差集。 迭代器遍历判断参数集合中是否包含目标集合中的元素，如果包含，则删除目标集合中的元素。

4. 修改
  - retainAll(Collection<?>): 集合求并集。迭代器遍历目标集合，判断只要元素不存在于参数集合中，则将元素从目标集合中移除。
  - clear()：通过迭代器移除集合中的元素。

5. 查询
  - contains(Object o)：通过迭代器进行循环判断集合中是否有某个元素，如果有则返回true。
  - containsAll(Collection<?>): 通过for循环皮判断参数中的元素，让每一个元素通过contains方法进行判断看，只要有一个元素不存在集合中，则返回false。

以上就是AbstractCollection类的讲解，其实着重对[toArray](/java/util/abstractcollection/M_toArray.md)方法进行了分析，了解了AbstractCollection类对集合的扩容方式。

# 总结
AbstractCollection抽象类是Collection接口唯一的实现类，主要对Collection接口进行了最小程度的实现。

# 抽象类和接口
除了源码分析，我们也可以来看看抽象类和接口这两个概念。

有句很经典的话：
> 抽象类是对一类事物的抽象，而接口是对一种行为的抽象。

## 二者设计上的区别
1. 举个例子：比如飞机和鸟是两个不同的类别，但是他们都有一个共性——飞行。那么在设计上，可以将飞机设计为一个类AirPlane,将鸟设计为一个Bird类，但是不能将飞行设计为一个类，因为飞行只是一个行为特性，并不能具体描述某一类事物。既然飞行是行为，那么我们可以将这一行为设计为一个接口Fly,它具有fly方法。然后AirPlane和Bird分别根据需要实现Fly这个接口，再进行各自的实现。至于有不同种类的飞机，比如如果是战斗机、民用飞机等直接继承Airplane即可，这里可以看出，继承是一个“是不是”的关系。而有些鸟是不能飞的，那不用实现Fly这个接口，所以这里可以看出，接口实现是“有没有”的关系。

2. 设计层面上不同，抽象类作为很多子类的父类，它是一种模板式设计。而接口是一种行为规范，它是一种辐射式设计。什么是模板式设计？大家都用过ppt里面的模板，如果用模板A设计了ppt B 和ppt C，ppt B和ppt C公共的部分就是模板A了，如果它们的公共部分需要改动，则只需要改动模板A就可以了，不需要重新对ppt B和ppt C进行改动。而辐射式设计，比如某个电梯都装了某种报警器，一旦要更新报警器，就必须全部更新。也就是说对于抽象类，如果需要添加新的方法，可以直接在抽象类中添加具体的实现，子类可以不进行变更；而对于接口则不行，如果接口进行了变更，则所有实现这个接口的类都必须进行相应的改动。

下面看一个网上流传最广泛的例子：门和警报的例子：门都有open( )和close( )两个动作，此时我们可以定义通过抽象类和接口来定义这个抽象概念：
```java
abstract class Door {
    public abstract void open();
    public abstract void close();
}
```
或者：
```java
interface Door {
    public abstract void open();
    public abstract void close();
}
```
但是现在如果我们需要门具有报警alarm( )的功能，那么该如何实现？下面提供两种思路：

1. 将这三个功能都放在抽象类里面，但是这样一来所有继承于这个抽象类的子类都具备了报警功能，但是有的门并不一定具备报警功能；
2. 将这三个功能都放在接口里面，需要用到报警功能的类就需要实现这个接口中的open( )和close( )，也许这个类根本就不具备open( )和close( )这两个功能，比如火灾报警器。

从这里可以看出，Door的open()、close()和alarm()根本就属于两个不同范畴内的行为，open()和close()属于门本身固有的行为特性，而alarm()属于延伸的附加行为。因此最好的解决办法是单独将报警设计为一个接口，包含alarm()行为,Door设计为单独的一个抽象类，包含open和close两种行为。再设计一个报警门继承Door类和实现Alarm接口。
```java
interface Alram {
    void alarm();
}

abstract class Door {
    void open();
    void close();
}

class AlarmDoor extends Door implements Alarm {
    void oepn() {
      //....
    }
    void close() {
      //....
    }
    void alarm() {
      //....
    }
}

```


---

参考文档：
1. [深入理解Java的接口和抽象类](https://www.cnblogs.com/dolphin0520/p/3811437.html)
2. [java提高篇（四）-----抽象类与接口](https://blog.csdn.net/chenssy/article/details/12858267)

