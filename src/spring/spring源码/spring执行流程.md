# Spring执行流程

想要弄清Spring的执行流程，首先要捋清几个类之间的关系，我们希望见名知意，而不需要刻意地去死记硬背。

我们要带着这些类的作用是什么，什么时候执行的思考去理解Spring的执行流程。大道至简。

总而言之，Spring的目的就是生成Bean对象。



* BeanFactroy：接口，对容器的**操作**
* DefaultListableBeanFactory：容器
* ApplicationConext：容器
* BeanDefinitionRegistry：顶层接口，定义了如何对 BeanDefinition 进行**管理**
* SimpleBeanDefinitionRegistry ： BeanDefinitionRegistry 的一个简单实现（可通过它来了解下具体实现）
* BeanDefinition：接口，定义了对Bean的属性设置和查询，诸如类名、scope、属性、构造函数参数列表、依赖的bean、是否单例类等. 
* FactoryBean：接口
* BeanPostProcessor
* BeanFactoryPostProcessor：提供扩展点操作bean的定义



### BeanDefinitionRegistry

> Interface for registries that hold bean definitions, for example RootBeanDefinition and ChildBeanDefinition instances. Typically implemented by BeanFactories that internally work with the AbstractBeanDefinition hierarchy.

顶层接口，定义了如何对 BeanDefinition 进行**管理**

```java
public interface BeanDefinitionRegistry extends AliasRegistry {

	void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
			throws BeanDefinitionStoreException;

	void removeBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;

	BeanDefinition getBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;

	boolean containsBeanDefinition(String beanName);

	String[] getBeanDefinitionNames();

	int getBeanDefinitionCount();

	boolean isBeanNameInUse(String beanName);

}
```



### StandardEnvironment

#  BeanFactory

> The root interface for accessing a Spring bean container

`BeanFactory`是一个接口，它是Spring中工厂的顶层规范，是SpringIoc容器的核心接口，它定义了`getBean()`、`containsBean()`等管理Bean的通用方法。Spring的容器都是它的具体实现如：

- DefaultListableBeanFactory
- XmlBeanFactory
- ApplicationContext

# BeanFactoryPostProcessor

可用于用户扩展

> Allows for custom modification of an application context's bean definitions,adapting the bean property values of the context's underlying bean factory.

```java
@FunctionalInterface
public interface BeanFactoryPostProcessor {

	/**
	 * Modify the application context's internal bean factory after its standard
	 * initialization. All bean definitions will have been loaded, but no beans
	 * will have been instantiated yet. This allows for overriding or adding
	 * properties even to eager-initializing beans.
	 * @param beanFactory the bean factory used by the application context
	 * @throws org.springframework.beans.BeansException in case of errors
	 */
	void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;

```

# FactoryBean

首先它是一个Bean，但又不仅仅是一个Bean。它是一个能生产或修饰对象生成的工厂Bean，类似于设计模式中的工厂模式和装饰器模式。它能在需要的时候生产一个对象，且不仅仅限于它自身，它能返回任何Bean的实例。

从它定义的接口可以看出，`FactoryBean`表现的是一个工厂的职责。   **即一个Bean A如果实现了FactoryBean接口，那么A就变成了一个工厂，根据A的名称获取到的实际上是工厂调用`getObject()`返回的对象，而不是A本身，如果要获取工厂A自身的实例，那么需要在名称前面加上'`&`'符号。**

```java
@Component
public class MyFactoryBean implements FactoryBean<Red> {
    @Override
    public Red getObject() throws Exception {return new Red();}

    @Override
    public Class<?> getObjectType() {return Red.class;}

    @Override
    public boolean isSingleton() {return true;}
}
```

执行invokeBeanFactoryPostProcessors之后，BeanDefinitionMap里没有Red的定义信息。而是执行finishBeanFactoryInitialization的时候放到单例池中



## BeanPostProcessor

Bean后置处理器？什么时候运行，作用是什么？

## ApplicationContextInitializer





### 循环依赖和三级缓存

先说明前提：**原型**(Prototype)的场景是**不支持**循环依赖的，通常会走到`AbstractBeanFactory`类中下面的判断，抛出异常。

```
if (isPrototypeCurrentlyInCreation(beanName)) {
  throw new BeanCurrentlyInCreationException(beanName);
}
复制代码
```

原因很好理解，创建**新的A**时，发现要注入**原型字段B**，又创建**新的B**发现要注入**原型字段A**...

这就套娃了, 你猜是先**StackOverflow**还是**OutOfMemory**？

Spring怕你不好猜，就先抛出了**BeanCurrentlyInCreationException**

基于构造器的循环依赖，就更不用说了，[官方文档](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-dependency-resolution)都摊牌了，你想让构造器注入支持循环依赖，是不存在的，不如把代码改了。

那么默认单例的属性注入场景，**Spring**是如何支持循环依赖的？





非常生动，我自己也研究过关于 Spring 解决依赖循环的方法，总的来说有两点： 1、利用缓存识别已经遍历过的节点； 2、利用Java引用，先提前设置对象地址，后完善bean；



spring内部有三级缓存：

- singletonObjects 一级缓存，用于保存实例化、注入、初始化完成的bean实例
- earlySingletonObjects 二级缓存，用于保存实例化完成的bean实例
- singletonFactories 三级缓存，用于保存bean创建工厂，以便于后面扩展有机会创建代理对象。



https://mp.weixin.qq.com/s/fX4ze5YuLI5cqVaZhxq1tw



### 生命周期

![image-20210113162521027](https://tva1.sinaimg.cn/large/008eGmZEly1gmm5agj2laj30zk09yn2a.jpg)



![image-20210113134229058](https://tva1.sinaimg.cn/large/008eGmZEly1gmm0l1z4n2j30vi0la48o.jpg)





循环依赖总结一下（假设A,B之间循环依赖）：
一级缓存singletonObject，也就是常说的单例池，是个Map
二级缓存earlySingletonObjects，也就是提前一点的单例池，哈哈，字面翻译,也是Map
三级缓存singletonFactories，这个Map有点特殊，因为这个Map的value存放的是一个lambda表达式
1、单例池不能存放原始对象，只能存放经过完整生命周期的对象，也就是java bean
2、A，B在注入属性都会执行一个addSingletonFactory方法，这个方法里面三级缓存就出现了，三级缓存put了key为beanName，value为一个lambda表达式
3、其实最容易绕晕的地方是，当B注入属性A的时候，执行populateBean注入一个bean的属性的时候会执行getSingleton这个方法，一定要记得！！populateBean方法体中没有直接调用getSingleton这个方法，但一定要记得，执行了这个方法
4、getSingleton这个方法，会依次到一级缓存，二级缓存，三级缓存中get(beanName)，很显然当B注入A属性的时候，一级，二级里面都没有内容，只有三级有，这时会执行lambda表达式，lambda表达式的作用就是生成代理对象！！然后把生成的代理对象存入二级缓存，并返回这个代理对象，B就会得到这个代理对象A，B就会认为这个代理对象A就是A的最后的bean对象，因此也就完成了对A的属性注入这步操作，接着依次执行B后续的操作，最后就完成了B的生命周期，B就成功变成了bean对象，B也就完成了使命
5、当B完成使命之后，A就会继续注入B，这时就会注入属性成功了，接下来开始执行AOP操作，因为上一步中A已经生成了代理对象A，也就是相当于完成了AOP，所以B就不执行AOP操作了，此时A就会执行最后一步操作，将代理对象A放入到单例池中去，这时A就会执行方法getSingleton，从二级缓存中获得了代理对象A，最后将其存入单例池，也就是一级缓存！好了，现在A和B都放入了单例池，圆满结束！！！！



## BeanDefinition

将类中所有的定义信息包装成对象放到BeanDefinitionMap中，后续要创建对象的时候，就不用再去解析xml或者注解，直接按照BeanDefinition去生成对应的bean。

BeanDefinition有哪些属性？



AnnotatedGenericBeanDefinition：以@Configuration注解标记的会解析为AnnotatedGenericBeanDefinition

ConfigurationClassBeanDefinition：以@Bean注解标记的会解析为ConfigurationClassBeanDefinition

ScannedGenericBeanDefinition：以@Component注解标记的会解析为ScannedGenericBeanDefinition

```java
public class Application {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
        RootBeanDefinition rootBeanDefinition = new RootBeanDefinition();
        rootBeanDefinition.setBeanClass(User.class);
        rootBeanDefinition.getPropertyValues().add("name", "liang");
        rootBeanDefinition.getPropertyValues().add("age", 12);
        ctx.registerBeanDefinition("user", rootBeanDefinition);
        User bean = ctx.getBean(User.class);  // 这一步去生成bean
        System.out.println(bean); //User(name=liang, age=12)
    }
}
```



https://www.jianshu.com/p/a6a03d94d6f7

https://cloud.tencent.com/developer/article/1497805

### BeanFactoryPostProcessor

接口，用来更改beandefinition的信息。

内置的BeanFactoryPostProcessor做了哪些事情，比如ConfigurationClassPostProcessor，扫描，将我们的类变成BeanDefinition；还有cglib

![image-20210114231835072](https://tva1.sinaimg.cn/large/008eGmZEly1gmnmusl290j31ao0degva.jpg)

### ConfigurationClassPostProcessor很重要

![image-20210114232322379](https://tva1.sinaimg.cn/large/008eGmZEly1gmnmzs7zxgj30so0m8tj4.jpg)

### ClassPathBeanDefinitionScanner

ctx.scan("com.zstu")

spring提供的这些reader和scanner是提供给程序员的api，方便操作内部



### @Configutation

如果加了@Configuration，那么是生成的是代理对象，不加的话就是普通的appConfig对象

```java
public class Application {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
        AppConfig bean = ctx.getBean(AppConfig.class);  // 这一步去生成bean
        System.out.println(bean);
    }
}
```

输出

```java
com.zstu.springdemo.config.AppConfig$$EnhancerBySpringCGLIB$$c580f573@2ed0fbae
```

此外

```java
//@Configuration
@ComponentScan(basePackages = "com.zstu.springdemo")
public class AppConfig {

    @Bean
    public User user(){
        return new User();
    }

    @Bean
    public Person person(){
        user();
        return new Person();
    }
}
```

如果不加@Configuration，User对象会被实例化两次；加了@Configuration，User会被实例化一次，内部还是代理对象起的作用。



### @Bean

对象的创建在finishBeanFactoryInitialization(beanFactory);

getBean获取bean，没有的话根据beandefinition里的factorymethod去创建

### @AutoWire

先更加类型去匹配注入对象，没有的话按照名字去匹配注入





