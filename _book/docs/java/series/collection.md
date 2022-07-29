# 伊始
阳春三月，草长莺飞,我很享受校园风光（比如说情人坡上的美女~），也常常会和小伙伴一起去旁听一些学校比较有意思的公开课。学校的阶梯教室、讲师的尊尊教导，常常会梦里萦绕（谨此怀念过去许久的美好校园时光）。

为了方便操作，我们常常会将同一类型的元素放在一个阶梯教室里，哦，就是放在一起。在java中，可能你马上会想到数组，确实，数组是一个很好的解决方案。

数组有个特点，它的底层实现是在内存中寻找一块连续的座位，然后把具有相同特点的元素放在一起。可是公开课，常常人满为患，我和小伙伴常常找不到一块连续的座位，如果我们非要一块连续的座位，那么很可能就会被告知OOM异常。或许某次我们非常幸运，找到了连着的座位，可是后来又来了几个小伙伴要一起旁听课,为了体现基友情，我们又要重新找连在一起的座位，重新创建数组，这样就很费精力......所以，我们常常分开坐。

为了解决类似上面的问题，java提供了令人非常激动的保存对象工具——集合。集合解决了“在任意时刻，任意位置，创建任意数量对象”的问题。

在本节，我们会去探索java中集合的实现，学习集合的使用，分析涉及的算法，看看用到的设计模式，体会设计者的设计思路。

以下是集合系列文章：

1. [I_Iterator](/java/util/I_iterator.md)
2. [I_Iterable](/java/lang/I_Iterable.md)
3. [I_Collection](/java/util/I_Collection.md)
4. [c_AbstractCollection](/java/util/I_AbstractCollection.md)
5. [I_ListIterator](/java/util/I_ListIterator.md)
6. [I_List](/java/util/I_List.md)
7. [c_AbstractList](/java/util/c_AbstractList.md)
8. [C_ArrayList](/java/util/C_ArrayList.md)
