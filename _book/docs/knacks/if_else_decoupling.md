# if...else...解耦

在spring的[初始化bean](/spring/bean/bean.md "初始化bean")中讲到，过深的if...else...条件判断使得代码变得可读性差、不易维护。

本文通过静态工厂+策略模式对if...else...实现解耦。

1.以[初始化bean](/spring/bean/bean.md "初始化bean")中发奖为例，首先定义一个接口：
```java
/**
 * @description: 发放奖励
 * @author: yongmingcode
 */
public interface IHandOutPrize {
    void doHandOut();
}

```

2.定义发放现金奖励类：
```java
/**
 * @description: 发放现金奖励
 * @author: yongmingcode
 */
@Service
public class HandOutCash implements IHandOutPrize, InitializingBean {
    public void doHandOut() {
        System.out.println(" -------HandOutCash doHandOut-------");
    }

    /** 注册 */
    public void afterPropertiesSet() throws Exception {
        HandOutPrizeFactory.registerHandOutPrize("cash", this);
    }
}

```

3.定义发放积分奖励类：
```java
/**
 * @description: 发放积分奖励
 * @author: yongmingcode
 */
@Service
public class HandOutPoint implements IHandOutPrize, InitializingBean {
    public void doHandOut() {
        System.out.println("-------------HandOutPoint  doHandOut -----------");
    }

    /** 注册 */
    public void afterPropertiesSet() throws Exception {
        HandOutPrizeFactory.registerHandOutPrize("point", this);
    }
}

```

4.定义发放奖品的静态工厂类：
```java
/**
 * @description: 发放奖品的静态工厂
 * @author: yongmingcode
 */
public class HandOutPrizeFactory {

    private static Map<String, IHandOutPrize> handOutPrizeMap = new HashMap<String, IHandOutPrize>();

    public HandOutPrizeFactory() {
    }

    private static IHandOutPrize EMPTY = new EmptyHandOutPrize();

    /** 注册 */
    public static void registerHandOutPrize(String prizeType, IHandOutPrize handOutPrize){
        handOutPrizeMap.put(prizeType, handOutPrize);
    }

    /** 获取 */
    public static IHandOutPrize getHandOutPrize(String prizeType){
        IHandOutPrize handOutPrize = handOutPrizeMap.get(prizeType);
        return handOutPrize == null ? EMPTY : handOutPrize;
    }

    private static class EmptyHandOutPrize implements IHandOutPrize{
        public void doHandOut() {
            System.out.println(" EmptyHandOutPrize ");
        }
    }
}

```

5.最后，写一个发放现金奖励类：
```java

/**
 * @description: if...else...解耦
 * @author: yongmingcode
 */
public class IfElseDecoupling {
    public static void main(String[] args){
        // 发放现金奖励
        handOutPrize("cash");
    }

    private static void handOutPrize(String prizeType){
        IHandOutPrize handOutPrize = HandOutPrizeFactory.getHandOutPrize(prizeType);
        handOutPrize.doHandOut();
    }
}

```
上面代码的解释：

1. 定义了一个发放奖励的IHandOutPrize接口，该接口只有一个发放奖励的方法；
2. HandOutCash、HandOutPoint是IHandOutPrize接口的两个实现类，同时实现了InitializingBean接口（该接口关介绍请移驾[bean的初始化方法](/spring/bean/bean_initialization_method.md)），并在afterPropertiesSet()方法中对当前类进行了注册；
3. 通过静态工厂HandOutPrizeFactory类进行奖励的注册和获取；
4. 在IfElseDecoupling类中进行发放奖励，如发放现金奖励只需要两行代码就够了。

总结：

优点：以上对if...else...形式代码进行了解耦重构，重构后的逻辑清晰、明了，如果新增一种奖品，只需要定义一个新的类即可，这样大大提高了代码的可读性和可维护性。

缺点：会增加很多类。

---

参考资料：

1. [代码重构：用工厂+策略模式优化过多的if else代码块](https://www.cnblogs.com/pfblog/p/7815238.html "代码重构：用工厂+策略模式优化过多的if else代码块")

2. [我的Java设计模式-策略模式](https://juejin.im/post/59facc30518825297a0e164c "我的Java设计模式-策略模式")

3. [Spring核心接口之InitializingBean](https://segmentfault.com/a/1190000012461362 "Spring核心接口之InitializingBean")
