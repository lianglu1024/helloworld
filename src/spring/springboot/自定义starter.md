# 自定义starter

我们知道Springboot是由一个个starter组成的，想要什么功能，导入对应的starter就可以了。例如想用redis作为缓存，那么直接导入下面的依赖包即可

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

事实上`spring-boot-starter-data-redis`这个依赖包没有任何的代码逻辑，仅仅是导入自动配置的依赖。因此我们想自定义starter，通常要编写两个模块，一个是导入自动配置依赖包，另一个就是具体的代码处理模块。

**注意：**这里说下`artifactId`的命名问题，Spring 官方 Starter通常命名为`spring-boot-starter-{name}`如 `spring-boot-starter-web`， Spring官方建议非官方Starter命名应遵循`{name}-spring-boot-starter`的格式。

## 自定义redis starter

1.新建maven父项目redis-starter，在父项目下新建两个子项目(module)redis-spring-boot-starter和redis-spring-boot-configure

2.删除redis-spring-boot-starter项目下的src目录，只保留pom.xml

3.在redis-spring-boot-starter的pom.xml下导入redis-spring-boot-configure依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>redis-starter</artifactId>
        <groupId>com.zstu</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <artifactId>redis-spring-boot-starter</artifactId>

    <dependencies>
        <dependency>
            <groupId>com.zstu</groupId>
            <artifactId>redis-spring-boot-configure</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>
</project>
```

4.在redis-spring-boot-configure中导入spring-boot-autoconfigure依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>redis-starter</artifactId>
        <groupId>com.zstu</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>redis-spring-boot-configure</artifactId>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure</artifactId>
            <version>2.3.0.RELEASE</version>
        </dependency>
    </dependencies>
</project>
```

5.开始编写配置信息和功能代码

* 定义配置类

```java
package com.zstu.redis;

import org.springframework.boot.context.properties.ConfigurationProperties;

/**
 * 通过@ConfigurationProperties注解来进行对应的属性的装配
 */
@ConfigurationProperties(prefix = "spring.my.redis")
public class RedisProperties {
    private String url;
    private String port;
    private String name;
    private String password;

    public String getUrl() {
        return url;
    }

    public void setUrl(String url) {
        this.url = url;
    }

    public String getPort() {
        return port;
    }

    public void setPort(String port) {
        this.port = port;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}

```

* 定义服务类

```java
package com.zstu.redis;

public class RedisTemplate {
    private String url;
    private String port;
    private String name;
    private String password;

    public RedisTemplate(String url, String port, String name, String password) {
        this.url = url;
        this.port = port;
        this.name = name;
        this.password = password;
        System.out.println("RedisTemplate构造方法执行");
    }

    public String get(String key){
        return key;
    }
}
```

* 创建自动化配置类

```java
package com.zstu.redis;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * 创建自动化配置类
 */
@Configuration
@EnableConfigurationProperties(RedisProperties.class)
public class RedisAutoConfigure {
    @Autowired
    private RedisProperties redisProperties;

    @Bean
    public RedisTemplate redisTemplate(){
        return new RedisTemplate(redisProperties.getUrl(),
                redisProperties.getPort(),
                redisProperties.getName(),
                redisProperties.getPassword());
    }
}
```

* 在resource目录下新建META-INF，在目录下新建spring.factories

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.zstu.redis.RedisAutoConfigure
```

目录布局如下：

![image-20210106200206260](https://tva1.sinaimg.cn/large/008eGmZEly1gme87s9ntoj30d40az3zj.jpg)

生成本地jar包

![image-20210106200242167](https://tva1.sinaimg.cn/large/008eGmZEly1gme88ep3v5j30a6092dg8.jpg)

## 测试

1.新建web工程，导入如下依赖包

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>2.3.0.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>com.zstu</groupId>
            <artifactId>redis-spring-boot-starter</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

    </dependencies>
```

2.application.properties

```
spring.myredis.url=127.0.0.1
spring.myredis.port=6369
spring.myredis.name=root
spring.myredis.password=123456
```

3.控制层新建HelloController.java

```java
package com.zstu.controller;

import com.zstu.redis.RedisTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {
    @Autowired
    private RedisTemplate redisTemplate;

    @GetMapping("/hello")
    public String hello(){
        return redisTemplate.get("hello redis");
    }
}
```

4.编写启动类并运行，页面输出hello redis