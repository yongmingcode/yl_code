# I_Collection
Collection接口是[Iterable](/java/lang/I_Iterable.md)接口的子接口，所以只要实现Collection接口的类，都拥有for-each循环的能力。同时，Collection接口还是List、Set和Queue接口的父接口，所以该接口中的方法既可以操作Set集合，也可以用于操作List和Queue集合。

# 方法
既然是接口，那么就是对行为的抽象。我将该接口中操作集合元素的行为分为以下类型：
1. 获得集合属性相关的方法：
    - size(): 返回集合中元素的数量。
    - isEmpty(): 判断集合是否有元素，如果没有返回true。

2. 操作集合相关方法：
    - iterator(): 返回操作该集合的迭代器。
    - toArray(): 返回一个包含集合所有元素的新创建的数组。
    - equals(Object): 比较指定对象与此集合的相等性。
    - hashCode(): 返回该集合的hash code值。
    - spliterator()：返回一个Spliterator迭代器。
    - stream()：返回一个以此集合为源的Stream对象。
    - parallelStream()： 返回一个以此集合为源的可能并行的Stream对象。

3. 添加：
    add(E): 向集合中添加一个元素，操作成功则返回true。
    - addAll(Collection<? extends E>): 向集合中添加另外一个集合的所有元素。成功则返回true。

4. 删除：
    - remove(Object): 删除集合中的某个元素，操作成功则返回true。
    - removeAll(Collection<?>): 删除本集合中指定集合所拥有的所有元素，成功则返回true。
    - removeIf(Predicate<? extends E>): 根据条件，删除集合中的元素，jdk1.8新增方法，相关介绍请移驾[M_removeIf](/java/util/M_removeIf.md)。

5. 修改：
    - retainAll(Collection<?>): 将当前集合中在指定集合没有的元素删除掉，保留在指定集合中有的元素。
    - clear(): 清空当前集合中的元素。

6. 查询：
    - contains(Object): 判集合中是否包含指定的元素，包含则返回true。
    - containsAll(Collection<?>): 判断集合是否包含另外一个集合的所有元素，是则返回true

以上就是Collection接口的所有方法。

作为一个接口，就是对行为的抽象，所以Collection接口中的方法都是操作集合的一些行为。而操作集合的行为，一般都少不了增删改查、获得集合属性等。

