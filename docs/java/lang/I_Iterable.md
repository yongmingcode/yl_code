# I_Iterable
该接口是集合的顶级接口，正如jdk中注释的那样，实现了该接口，则可以进行for-each循环。

# 方法
1. iterator：该方法返回一个泛型类型的迭代器。
2. forEach：jdk1.8新增方法，对集合中每个元素遍历执行给定的操作。
3. spliterator：jdk1.8新增方法，该方法提供了一个可以并行遍历元素的迭代器，以适应现在cpu多核时代并行遍历的需求。更多相关介绍请查看文末参考资料。

# 方法探究

## iterator方法
Iterable接口既然能让它的实现类能够进行for-each循环，那我们就拿ArrayList类来做个小示例（ArrayList是Iterable接口的子孙接口）：
```java
import java.util.*;
public class Test{
	public static void main(String[] args){
		List<String> list = new ArrayList<String>();
		list.add("a");
		list.add("b");
		list.add("c");
		for (String str : list) {
			System.out.println(str);
		}
	}
}
```
编译运行后控制台输出：
```java
> javac Test.java
> java Test
a
b
c
```

那for-each到底是怎么进行循环的呢？我们通过javap命令对字节码进行反编译，看看能不能得出答案。

反编译后的字节码输出如下：
```java
  public static void main(java.lang.String[]);
    Code:
       0: new           #2                  // class java/util/ArrayList
       3: dup
       4: invokespecial #3                  // Method java/util/ArrayList."<init>":()V
       7: astore_1
       8: aload_1
       9: ldc           #4                  // String a
      11: invokevirtual #5                  // Method java/util/ArrayList.add:(Ljava/lang/Object;)Z
      14: pop
      15: aload_1
      16: ldc           #6                  // String b
      18: invokevirtual #5                  // Method java/util/ArrayList.add:(Ljava/lang/Object;)Z
      21: pop
      22: aload_1
      23: ldc           #7                  // String c
      25: invokevirtual #5                  // Method java/util/ArrayList.add:(Ljava/lang/Object;)Z
      28: pop
      29: aload_1
      30: invokevirtual #8                  // 注释1 Method java/util/ArrayList.iterator:()Ljava/util/Iterator;
      33: astore_2
      34: aload_2
      35: invokeinterface #9,  1            // 注释2  InterfaceMethod java/util/Iterator.hasNext:()Z
      40: ifeq          63
      43: aload_2
      44: invokeinterface #10,  1           // 注释3 InterfaceMethod java/util/Iterator.next:()Ljava/lang/Object;
      49: checkcast     #11                 // class java/lang/String
      52: astore_3
      53: getstatic     #12                 // Field java/lang/System.out:Ljava/io/PrintStream;
      56: aload_3
      57: invokevirtual #13                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      60: goto          34
      63: return
```
果然，从字节码文件我标注的注释1、2、3可以看出，其实for-each最终还是被jvm通过调用Iterator迭代器实现的。

## forEach方法
相信细心的小伙伴已经发现，forEach方法和[I_Iterator](/java/util/I_iterator.md)中的forEachRemaining方法，二者都是对元素遍历执行给定的操作,那为何会有这2个方法呢？这不是多余吗？

其实二者有所不同：
1. forEach方法使用的是for-each循环遍历，而forEachRemaining方法使用迭代器Iterator遍历元素。
2. forEach方法是对Iterable中的所有元素进行遍历，可以多次调用；forEachRemaining方法第二次调用不会做任何操作，因为不会有下一个元素。

## spliterator方法
我们看一下该方法的源码：
```java
default Spliterator<T> spliterator() {
        return Spliterators.spliteratorUnknownSize(iterator(), 0);
}
```
里面只有一行代码，调用了Spliterators的spliteratorUnknownSize方法，继续往下看：
```java
public static <T> Spliterator<T> spliteratorUnknownSize(Iterator<? extends T> iterator,
                                                            int characteristics) {
        return new IteratorSpliterator<>(Objects.requireNonNull(iterator), characteristics);
}

```
该方法通过有参构造器创建了一个IteratorSpliterator对象并返回，我们看这个构造器做了什么：
```java
public IteratorSpliterator(Iterator<? extends T> iterator, int characteristics) {
            this.collection = null;
            this.it = iterator;
            this.est = Long.MAX_VALUE;
            this.characteristics = characteristics & ~(Spliterator.SIZED | Spliterator.SUBSIZED);
}
```
这个构造器其实就是使用给定的迭代器创建了Spliterator遍历器，并给定了初始大小。

而Spliterator是什么呢？Spliterator即splitable iterator可分割迭代器，它是一个接口，是Java为了并行遍历数据源中的元素而设计的迭代器，这个可以类比最早Java提供的顺序遍历迭代器Iterator，但一个是顺序遍历，一个是并行遍历。
> Spliterator接口用到了Fork/Join设计思想，总得来说就是用递归的方式把并行的任务拆分成更小的子任务，然后把每个子任务的结果合并起来生成整体结果。这里对此不做深究，以后会专门对Fork-Join进行讨论。

---

参考资料：

1. [Java8里面的java.util.Spliterator接口有什么用？](Java8里面的java.util.Spliterator接口有什么用？)
