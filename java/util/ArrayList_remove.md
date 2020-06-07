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

### 改进
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
程序的世界就是那么让人兴奋。后来,我又认识了“A new girl”——for-each，她可以提供更加方便的遍历操作。直接上...代码：
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
\- -！比上次还严重，程序直接抛异常：java.util.ConcurrentModificationException。I did not do anything......拿出我的尚方宝剑：javap，通过反编译发现，原来for-each循环是通过调用Iterator迭代器实现的啊，
