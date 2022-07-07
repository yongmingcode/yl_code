# Deque介绍
Deque是double ended queue，即双端队列的意思。允许元素从队列的两端向队列中添加、删除元素。

# 方法介绍
Deque接口中的方法一共有一下四部分：
## Deque定义的方法
- (add\offer\remove\poll\get\peek)First或Last从双端操作元素。
- removeFirstOccurrence(Object o)：从双端队列头部移除第一次出现的指定元素。
-  removeLastOccurrence(Object o)：从双端队列尾部移除第一次出现的指定元素。
-  descendingIterator()：返回一个反转的双端队列Iterator迭代器。

## 继承自Queue的方法
add、offer、remove、poll、element、peek这些方法与Queue中定义的一样，这里不再赘述。

## Stack栈方法
- void push(E e)： 入栈，此方法等效于addFirst()。
- E pop()：出栈，此方法等效于 removeFirst()。

## 继承自Collection的方法
remove、contains、size、iterator方法。

# 小结
向文章开头介绍的那样，Deque定义了对双端队列从两端操作队列中元素的方法，这些方法中的每一种都以两种形式存在：一种在操作失败时引发异常，另一种返回特殊值null或false，具体取决于操作。
