# I_Iterator
Iterator顾名思义就是迭代器，这是自jdk1.2起就存在的专门用于集合的迭代器参数化接口。

# 用法举例

```java
import java.util.ArrayList;
import java.util.Iterator;

public class Test {
  public static void main(String[] paramArrayOfString) {
    ArrayList<String> arrayList = new ArrayList();
    arrayList.add("a");
    arrayList.add("b");
    arrayList.add("c");
    Iterator<String> iterator = arrayList.iterator();
    while (iterator.hasNext()) {
      String str = iterator.next();
      System.out.println(str);
    }
  }
}
```
运行结果：
```java
> java Test
a
b
c
```
上面定义了一个ArrayList，然后通过iterator对ArrayList进行了遍历。

# 接口方法
1. hasNext()：判断集合是否有下一个元素
2. next()：返回集合的下一个元素
3. remove()：移除集合中的某一个元素
4. forEachRemaining(Consumer<? super E> action)：jdk1.8新增方法，对集合中剩余遍历的元素执行给定的操作（action）

迭代器Iterator对集合中的元素挨个操作，来实现循环遍历。所以Iterator更像是指针的一种抽象，通过不断的移动指针的指向，来遍历内存中的数据。但是指针和迭代器确实不是同一概念，这里只是为了更加形象的描述迭代器（二者有本质的不同：比如迭代器返回的是对象的引用而不是对象的值、指针能够指向函数而迭代器不能。）

> 在jdk1.8之前：接口中的成员一律默认是 public 类型。接口中的变量默认被指定为public static final类型；接口中的方法默认时public abstract类型的抽象方法，而我们都知道抽象方法不能拥有方法体。

> 我们注意到remove和forEachRemaining方法都有方法体，这是为什么呢？jdk1.8对接口进行了增强，该版本针对接口新增了2大特性：1.static方法，2.default修饰符。我们注意到remove和forEachRemaining方法都是通过default修饰的，所以才使得它们能够拥有方法体。所以，jdk1.8中，如果要在接口中定义有方法体的方法，可以通过使用static或者是default关键字对方法进行修饰。

---

参考资料：

1. [迭代器和指针的区别](https://www.zhihu.com/question/54047747 "迭代器（iterator）和指针（pointer）区别在哪？")
2. [Java8接口增强](https://blog.csdn.net/Mr_dsw/article/details/83183191 "Java 8 特性——interface 中的 static 方法和 default 方法")
3. [深入理解Java的接口和抽象类](https://www.cnblogs.com/dolphin0520/p/3811437.html "https://www.cnblogs.com/dolphin0520/p/3811437.html")
