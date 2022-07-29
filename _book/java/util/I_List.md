# I_List
List接口是Collection接口的子接口，代表集合家族中有序、可重复的集合。集合中的每一个元素，都有其对应的索引。

# 方法
List接口既然是接口，那么就是对行为的抽象。同时它还是Collection接口的子接口,那么它当然拥有Collection接口里的全部方法。所以在Collection接口里有的方法，这里有不再赘述了。

1. 操作：
    - replaceAll(UnaryOperator<E>): 根据UnaryOperator指定的计算规则重新设置List集合的所有元素。比如根据集合中元素的长度重新来作为新的集合。
    - sort(Comparator<? super E>): 根据Comparator参数对List集合进行排序。源码中其实时调用了Arrays.sort()方法进行的排序，然后通过ListIterator迭代器进行设置集合元素。Arrays和ListIterator后续章节会进行讲解。比如根据集合中元素的长度重新排序。

2. 添加：
    - add(int, E):在指定位置插入指定的元素，该位置后面的元素索引全部加1。
    - addAll(int, Collection<? extends E>): 在指定位置插入指定的集合元素，增加后院元素的索引。

3. 删除：
    - remove(Object): 该方法在Collection中定义，List集合删除集合中的某个元素，判断集合中的元素是不是目标元素，是根据equals()方法进行判断的。
    - remove(int): 删除指定位置的元素。

4. 修改：
    - set(int, E): 将指定位置修改为指定的元素。

5. 查询：
    - get(int): 获得指定位置的元素。
    - indexOf(Object): 返回指定元素在List中第一次出现的位置索引。
    - lastIndexOf(Object): 返回指定元素在List中最后一次出现的位置索引。
    - subList(int,int): 获得从指定的开始位置到结束位置所有集合元素组成的子集合。

6. 属性：
    - listIterator()：获得列表元素的列表迭代器。
    - listIterator(int): 从列表中指定的位置开始，返回列表中元素的列表迭代器。

从上面可以看出，相比于Collection接口，List集合里增加了一些根据索引来操作集合元素的方法。

> 有个很有意思的问题：既然List接口是Collection接口的子接口，那么它当然拥有Collection接口里的全部方法，可是为何在List接口中又重新定义了size、isEmpty等方法呢？我们可以想像一下，当jvm发现几个类之间时继承关系，如果要使用顶级父类的方法，是不是还要一个类一个类的去调用呢？喝口咖啡，当然不会允许这样！为了避免一层层向父级调用，所以在子类中也会同样得定义父类拥有的方法。为了提高性能，java的全力以赴可见一斑。

---

参考资料：

1. [《疯狂Java讲义》]()
