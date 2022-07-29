# 介绍
ListIterator接口是[Iterator](/java/util/I_Iterator.md)接口的子接口，所以拥有Iterator接口的所有方法。而它的作用也像它的名字一样简单直接，就是只能用于List及其子类的迭代器。

# 方法
这里只介绍其相对其父类[Iterator](/java/util/I_Iterator.md)独有的方法展开介绍。

1. 查询：
  - hasPrevious(): 是否拥有上一个元素。
  - previous(): 获得列表中的上一个元素。
  - nextIndex(): 获得下一个元素的坐标。
  - previousIndex(): 获得上一个元素的坐标。

2. 修改：
  - set(E): 将next()或者previous()返回的元素替换成指定的元素。

3. 添加：
  - add(E): 将指定的元素添加到列表中。添加的位置是next()返回的元素（如果有的话）之前，或者是previous()返回的元素（如果有的话）之后。

从上面方法中可以看出，ListIterator接口增加了从后向前遍历列表的功能，在[I_List](/java/util/List.md)接口中我们讲到,List集合中，每一个元素都有其对应的坐标，所以这里也增加了获取坐标的方法。
