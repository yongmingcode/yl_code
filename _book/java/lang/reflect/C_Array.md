# C_Array
java数组类，里面定义了很多静态方法供程序员创建和使用数组。这些方法除了构造方法和获得实例的方法外，全都是native方法。

# 方法解读

## newInstance(Class<?>, int): 创建一个指定长度、类型的数组。
```java
public static Object newInstance(Class<?> componentType, int length) throws NegativeArraySizeException {
    return newArray(componentType, length);
}

private static native Object newArray(Class<?> componentType, int length) throws NegativeArraySizeException;
```
newInstance方法内部调用了原生native方法newArray，返回一个Object。*该Object实质为数组类型*(可以通过：返回值.getClass()方法查看所属类型)。
    > newArray为本地原生方法，由虚拟机实现。







