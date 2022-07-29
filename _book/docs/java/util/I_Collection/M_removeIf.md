# M_removeIf方法解读
该方法使1.8新增方法，使用该方法，可以根据条件删除集合中的元素。

# 例子
1.7 & 1.8 删除集合中符合条件的元素。
## jdk 1.7 删除集合中为"b"的元素

```java
import java.util.*;

public class Test{
	public static void main(String[] args){
		List<String> list = new ArrayList<String>();
		list.add("a");
		list.add("b");
		list.add("c");
		System.out.print(list);
		System.out.println();
		Iterator<String> itr = list.iterator();
		while(itr.hasNext()){
			if("b".equals(itr.next())){
				itr.remove();
			}
		}
		System.out.print(list);
	}
}
```
上面的代码利用Iterator迭代器来删除list中为“b”的元素。来看看执行结果：
```java
> javac Test.java
> java Test
[a, b, c]
[a, c]
```
从结果可以看出，list中为“b”的元素成功被删除了。

## jdk 1.8 删除集合中为"b"的元素
```java
import java.util.*;

public class Test{
	public static void main(String[] args){
		List<String> list = new ArrayList<String>();
		list.add("a");
		list.add("b");
		list.add("c");

		System.out.print(list);
		System.out.println();

		list.removeIf(str -> "b".equals(str));

		System.out.print(list);
	}
}
```
上面的代码通过jdk 1.8来删除list中为“b”的元素。我们来看一下结果：
```java
> javac Test.java
> java Test
[a, b, c]
[a, c]
```
显然，删除成功了。这也是为什么我们需要升级技术的一个原因吧。
