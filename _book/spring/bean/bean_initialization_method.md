# bean的初始换方法
Spring的Bean在初始化完成后及销毁之前，允许程序执行特定的操作。有一下三种方式定指定：
- 实现InitializingBean/DisposableBean接口来指定初始化之后/销毁之前的操作方法
- 通过配置文件的<bean>标签中init-method/destroy-method属性指定初始化之后/销毁之前调用的操作方法
- 在指定方法上加上@PostConstruct或@PreDestroy注解来指定该方法是在初始化之后还是销毁之前调用

通过下面代码来验证：

配置文件spring-context.xml：
```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

	<!-- 包扫描器，扫描带Spring注解的类 -->
	<context:annotation-config/>

	<bean id="initAndDestroySeqBean" class="com.init.TestInitializationBean" init-method="initMethod" destroy-method="destroyMethod"/>
</beans>

```


测试类：
```java
package com.init;

import org.springframework.beans.factory.DisposableBean;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

/**
 * @author: yongmingcode
 * @create: 2020-04-22 17:43
 */
public class TestInitializationBean implements InitializingBean, DisposableBean {

    public TestInitializationBean() {
        System.out.println("----------------------TestInitializationBean 构造方法执行------------------");
    }

    /** 实现InitializingBean/DisposableBean接口来指定初始化之后/销毁之前的操作方法 */
    public void afterPropertiesSet() throws Exception {
        System.out.println("TestInitializationBean afterPropertiesSet ,表示实现InitializingBean接口初始化之后执行");
    }

    public void destroy() throws Exception {
        System.out.println("TestInitializationBean destroy ,表示实现DisposableBean接口销毁之前执行");
    }

    /** 通过配置文件的<bean>标签中init-method/destroy-method属性指定初始化之后/销毁之前调用的操作方法 */
    public void initMethod(){
        System.out.println("TestInitializationBean initMethod ,表示配置init-method属性初始化之后执行");
    }

    public void destroyMethod(){
        System.out.println("TestInitializationBean destroyMethod ,表示配置destroy-method属性销毁之前执行");
    }

    /** 在指定方法上加上@PostConstruct或@PreDestroy注解来指定该方法是在初始化之后还是销毁之前调用 */
    @PostConstruct
    public void postConstruct(){
        System.out.println("TestInitializationBean postConstruct ,表示@PostConstruct注解初始化之后执行");
    }

    @PreDestroy
    public void preDestroy(){
        System.out.println("TestInitializationBean preDestroy ,表示@PreDestroy注解销毁之前执行");
    }

    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("spring-context.xml");
        System.out.println("------------------------初始化完成------------------------------");
        context.close();
        System.out.println("------------------------销毁完成------------------------------");
    }
}


```

执行后控制台输出：
```java
2020-04-22 18:06:26 [main] [DefaultListableBeanFactory.getSingleton()] DEBUG - Creating shared instance of singleton bean 'initAndDestroySeqBean'
2020-04-22 18:06:26 [main] [DefaultListableBeanFactory.createBean()] DEBUG - Creating instance of bean 'initAndDestroySeqBean'
----------------------TestInitializationBean 构造方法执行------------------
2020-04-22 18:06:26 [main] [CommonAnnotationBeanPostProcessor.doWith()] DEBUG - Found init method on class [com.init.TestInitializationBean]: public void com.init.TestInitializationBean.postConstruct()
2020-04-22 18:06:26 [main] [CommonAnnotationBeanPostProcessor.doWith()] DEBUG - Found destroy method on class [com.init.TestInitializationBean]: public void com.init.TestInitializationBean.preDestroy()
2020-04-22 18:06:26 [main] [CommonAnnotationBeanPostProcessor.checkConfigMembers()] DEBUG - Registered init method on class [com.init.TestInitializationBean]: org.springframework.beans.factory.annotation.InitDestroyAnnotationBeanPostProcessor$LifecycleElement@a1ab9f17
2020-04-22 18:06:26 [main] [CommonAnnotationBeanPostProcessor.checkConfigMembers()] DEBUG - Registered destroy method on class [com.init.TestInitializationBean]: org.springframework.beans.factory.annotation.InitDestroyAnnotationBeanPostProcessor$LifecycleElement@a27dd7d7
2020-04-22 18:06:26 [main] [DefaultListableBeanFactory.doCreateBean()] DEBUG - Eagerly caching bean 'initAndDestroySeqBean' to allow for resolving potential circular references
2020-04-22 18:06:26 [main] [CommonAnnotationBeanPostProcessor.invokeInitMethods()] DEBUG - Invoking init method on bean 'initAndDestroySeqBean': public void com.init.TestInitializationBean.postConstruct()
TestInitializationBean postConstruct ,表示@PostConstruct注解初始化之后执行
2020-04-22 18:06:26 [main] [DefaultListableBeanFactory.invokeInitMethods()] DEBUG - Invoking afterPropertiesSet() on bean with name 'initAndDestroySeqBean'
TestInitializationBean afterPropertiesSet ,表示实现InitializingBean接口初始化之后执行
2020-04-22 18:06:26 [main] [DefaultListableBeanFactory.invokeCustomInitMethod()] DEBUG - Invoking init method  'initMethod' on bean with name 'initAndDestroySeqBean'
TestInitializationBean initMethod ,表示配置init-method属性初始化之后执行
2020-04-22 18:06:26 [main] [DefaultListableBeanFactory.createBean()] DEBUG - Finished creating instance of bean 'initAndDestroySeqBean'
2020-04-22 18:06:26 [main] [DefaultListableBeanFactory.doGetBean()] DEBUG - Returning cached instance of singleton bean 'org.springframework.context.event.internalEventListenerFactory'
2020-04-22 18:06:26 [main] [ClassPathXmlApplicationContext.initLifecycleProcessor()] DEBUG - Unable to locate LifecycleProcessor with name 'lifecycleProcessor': using default [org.springframework.context.support.DefaultLifecycleProcessor@1ef3efa8]
2020-04-22 18:06:26 [main] [DefaultListableBeanFactory.doGetBean()] DEBUG - Returning cached instance of singleton bean 'lifecycleProcessor'
2020-04-22 18:06:26 [main] [PropertySourcesPropertyResolver.getProperty()] DEBUG - Could not find key 'spring.liveBeansView.mbeanDomain' in any property source
------------------------初始化完成------------------------------
2020-04-22 18:06:26 [main] [ClassPathXmlApplicationContext.doClose()] INFO  - Closing org.springframework.context.support.ClassPathXmlApplicationContext@41f69e84: startup date [Wed Apr 22 18:06:25 CST 2020]; root of context hierarchy
2020-04-22 18:06:26 [main] [DefaultListableBeanFactory.doGetBean()] DEBUG - Returning cached instance of singleton bean 'lifecycleProcessor'
2020-04-22 18:06:26 [main] [DefaultListableBeanFactory.destroySingletons()] DEBUG - Destroying singletons in org.springframework.beans.factory.support.DefaultListableBeanFactory@19b93fa8: defining beans [org.springframework.context.annotation.internalConfigurationAnnotationProcessor,org.springframework.context.annotation.internalAutowiredAnnotationProcessor,org.springframework.context.annotation.internalRequiredAnnotationProcessor,org.springframework.context.annotation.internalCommonAnnotationProcessor,org.springframework.context.event.internalEventListenerProcessor,org.springframework.context.event.internalEventListenerFactory,initAndDestroySeqBean]; root of factory hierarchy
2020-04-22 18:06:26 [main] [CommonAnnotationBeanPostProcessor.invokeDestroyMethods()] DEBUG - Invoking destroy method on bean 'initAndDestroySeqBean': public void com.init.TestInitializationBean.preDestroy()
TestInitializationBean preDestroy ,表示@PreDestroy注解销毁之前执行
2020-04-22 18:06:26 [main] [DisposableBeanAdapter.destroy()] DEBUG - Invoking destroy() on bean with name 'initAndDestroySeqBean'
TestInitializationBean destroy ,表示实现DisposableBean接口销毁之前执行
2020-04-22 18:06:26 [main] [DisposableBeanAdapter.invokeCustomDestroyMethod()] DEBUG - Invoking destroy method 'destroyMethod' on bean with name 'initAndDestroySeqBean'
TestInitializationBean destroyMethod ,表示配置destroy-method属性销毁之前执行
------------------------销毁完成------------------------------
Disconnected from the target VM, address: '127.0.0.1:61729', transport: 'socket'


```

从执行结果可以看出：

Bean在实例化的过程中：Constructor > @PostConstruct >InitializingBean > init-method

Bean在销毁的过程中：@PreDestroy > DisposableBean > destroy-method

---

参考资料：

1. [Spring容器中的Bean几种初始化方法和销毁方法的先后顺序](https://blog.csdn.net/caihaijiang/article/details/8629725 "Spring容器中的Bean几种初始化方法和销毁方法的先后顺序")

2. [spring的Bean初始化方法的2种方式](https://blog.csdn.net/john1337/article/details/83614599 "spring的Bean初始化方法的2种方式")
