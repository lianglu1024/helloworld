# Mybatis与Spring整合

```java
UserMapper mapper = sqlSession.getMapper(UserMapper.class);
List<User> user = mapper.getUser();
```

UserMapper是一个接口，mybatis会帮我们动态生成一个代理对象mapper，用来实现接口里的所有方法（增删改查）。那mybatis是如何整合到spring中去的呢？说具体点就是如何将生成的动态代理对象mapper作为bean放到spring容器中去的呢？

## 整合

```java
@Component
public class UserService {

    @Autowired
    public UserMapper userMapper;
    ...
}
```

UserMapper是一个接口，那么mybaits会利用动态代理生成一个代理对象赋值给userMapper

将对象作为bean放到spring容器中的三种方式：

* FactoryBean的getObject方法创建一个对象
* @Bean
* BeanFactory的registerSingleton方法

在MyBatis中，就是利用的FactoryBean

```java
package com.zstu.springexplore.service;

import com.zstu.springexplore.mapper.UserMapper;
import org.springframework.beans.factory.FactoryBean;
import org.springframework.stereotype.Component;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

@Component
public class MyFactoryBean implements FactoryBean {
    @Override
    public Object getObject() throws Exception {
        Object instance = Proxy.newProxyInstance(MyFactoryBean.class.getClassLoader(), new Class[]{UserMapper.class}, new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                return null;
            }
        });
        System.out.println(instance);
        return instance;
    }

    @Override
    public Class<?> getObjectType() {
        return UserMapper.class;
    }
}
```

验证

```java
public class Test {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);

        Object myFactoryBean = ctx.getBean("myFactoryBean");
        
        UserService userService = ctx.getBean("userService", UserService.class);
        System.out.println(userService.userMapper);
    }
}
```

myFactoryBean的bean对象其实是代理对象，类型是UserMapper而非MyFactoryBean。

userMapper指代的就是代理对象

## 进一步整合

如果我们有多个Mapper，那岂不是要写多个FactoryBean来生成对应的代理对象喽。

duck不必！

第一步：定义一个类继承ImportBeanDefinitionRegistrar接口

```java
public class MyBeanDefinitionRegistarar implements ImportBeanDefinitionRegistrar {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {

        // 写死，后面可以通过注解的方式扫描mapper包下的所有类
        List<Class> mapperList = new ArrayList<>();
        mapperList.add(UserMapper.class);

        for (Class mapper : mapperList) {
            AbstractBeanDefinition beanDefinition = BeanDefinitionBuilder.genericBeanDefinition().getBeanDefinition();
            beanDefinition.setBeanClass(MyFactoryBean.class);
            beanDefinition.getConstructorArgumentValues().addGenericArgumentValue(mapper);
            registry.registerBeanDefinition(mapper.getName(), beanDefinition);
        }
    }
}
```

第二步：在AppConfig添加注解@Import，有了这个注解就会去执行相应的添加Definition逻辑

```java
@ComponentScan("com.zstu.springexplore")
@Import(MyBeanDefinitionRegistarar.class)
public class AppConfig {
}
```

注意：MyFactoryBean不再需要添加@Component注解了

**实现的原理就是我们主动的去生成BeanDefinition**

