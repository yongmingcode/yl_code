# C_Class
Class类我理解为:java运行时类的类。

这个描述可能有点奇怪，可以暂时把它认为是描述java类的类。这也很好的体现了：万事万物皆对象。

# 方法

1. getComponentType():它是一个native方法.
    ```java
    public native Class<?> getComponentType();
    ```
该方法的作用是*返回一个数组的组件类型，如果该类不是数组，则返回null*。那什么是数组的组件呢？就是数组元素！换句话说，就是返回数组中元素的类型。举例：
    ```java
    String[] o = {"aa", "bb", "cc"};
    System.out.println(o.getClass().getComponentType());
    控制台输出：
    class java.lang.String
    ```

