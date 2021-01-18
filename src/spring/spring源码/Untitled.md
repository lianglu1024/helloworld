## BeanFactoryPostProcessor

在BeanFactory标准初始化之后调用，来定制和修改BeanFactory的内容

所有的bean定义已经保存加载到beanFactory，但是bean的实例还未创建

##BeanDefinitionRegistryPostProcessor

继承自BeanFactoryPostProcessor，在所有bean定义信息将要被加载，bean实例还未创建。

早于BeanFactoryPostProcessor执行

作用：给容器中再额外的添加组件

```java
public class MyBeanDefinitionRegistryPostProcessor implements BeanDefinitionRegistryPostProcessor {
    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
        RootBeanDefinition redBeanDefinition = new RootBeanDefinition(Red.class);
        // 注册red组件
        registry.registerBeanDefinition("red", redBeanDefinition);
    }

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
    }
}
```

## ApplicationListener

监听容器中发布的事件。事件驱动模型开发

ApplicationContext事件机制是**观察者设计模式**的实现，通过ApplicationEvent类和ApplicationListener接口，可以实现ApplicationContext事件处理

简单来说就是：当向容器中发布某件事的时候，监听器会监听到事件的发生并执行相应的逻辑

自定义事件EatEvent

```java
public class EatEvent extends ApplicationEvent {
    private String time;
    private String food;

    public EatEvent(Object source, String time, String food) {
        super(source);
        this.time = time;
        this.food = food;
    }
}
```

实现对EatEvent事件的监听器

```java
@Component
public class EatApplicationListener implements ApplicationListener<EatEvent> {
    @Override
    public void onApplicationEvent(EatEvent event) {
        System.out.println("收到事件" + event.toString());
    }
}
```

发布事件

```java
public class Application {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
        ctx.publishEvent(new EatEvent("eat event", "12:00", "红烧肉"));
        ctx.close();
    }
}
```

原理很简单：当执行publishEvent方法的时候，内部会去找对应的ApplicationListener（即实现了`ApplicationListener<EatEvent>`接口的类），并执行onApplicationEvent方法。典型的观察者者模式。

`ApplicationEventMulticaster`：接口，由能够管理多个{@link ApplicationListener}对象并向它们发布事件的对象实现。

### @EventListener

我们可以使用@EventListener注解轻松完成事件监听的过程

```java
package com.zstu.springdemo.service;

import com.zstu.springdemo.event.EatEvent;
import org.springframework.context.ApplicationEvent;
import org.springframework.context.event.EventListener;
import org.springframework.stereotype.Component;

@Component
public class UserService {

    @EventListener(value = {EatEvent.class})
    public void listen(ApplicationEvent event){
        System.out.println("UserService收到事件" + event);
    }
}
```

当执行ctx.publishEvent(new EatEvent("eat event", "12:00", "红烧肉"))的时候，listen方法会执行

z

