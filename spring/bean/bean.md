# 伊始
五一、双十一、双十二之类的活动，常常需要根据用户获得的奖品，做不同的处理，这样代码中很可能就会有不少的判断。

比如活动中的奖品有：现金、积分、优惠券、BD奖品等，如果用户获得现金，我们就要做现金的逻辑处理，如果获得积分就做积分逻辑的处理。我们可能会这么做：

```java
    if("cash".equals(prize)){
        System.out.println("给用户发现金！");
    }else if("points".equals(prize)){
        System.out.println("给用户发积分！");
    }else if("coupons".equals(prize)){
        System.out.println("给用户发优惠券！");
    }else if("aiqiyi".equals(prize)){
        System.out.println("给用户发爱奇艺会员！");
    }
```
如果奖品很多，就进行很多个判断。不难发现，这样的代码可读性差、不易维护。我们可以使用策略模式优化它，优化demo会在后面的文章中更新。

这里可以通过org.springframework.beans.factory.InitializingBean接口来实现。而InitializingBean接口就是spring中bean的初始化方法之一。
