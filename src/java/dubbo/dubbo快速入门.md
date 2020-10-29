### Dubbo的服务架构

![](https://tva1.sinaimg.cn/large/00831rSTly1gd8ffrms9bj30tg0mwta8.jpg)

* 服务提供者将服务注册到注册中心
* 消费者从注册中心获取提供者的地址
* 消费者连接提供者获取数据



### 快速入门

#### 可视化web界面

1、安装zookeeper

2、下载[dubbo-admin](https://github.com/apache/dubbo-admin)，并用IDEA打开运行

3、访问127.0.0.1:7001(root/root)

![](https://tva1.sinaimg.cn/large/00831rSTly1gd8fug0nabj31hc0hn420.jpg)

#### 服务提供者

1、新建Spring maven项目，并在pom中添加dubbo和zookeeper依赖

```
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>2.7.3</version>
        </dependency>

        <dependency>
            <groupId>com.github.sgroschupf</groupId>
            <artifactId>zkclient</artifactId>
            <version>0.1</version>
        </dependency>

        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>2.12.0</version>
        </dependency>

        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.4.14</version>
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
```

2、在application.properties中配置

```
server.port=8001

dubbo.application.name=dubbo_server

dubbo.registry.address=zookeeper://127.0.0.1:2181

dubbo.scan.base-packages=com.example.dubbo_server.server
```

3、编写接口和实现类

TicketServer.java

```java
package com.example.dubbo_server.server;

public interface TicketServer {

    public String getTicket();
}
```

TicketServerImpl.java

```java
package com.example.dubbo_server.server;

import org.apache.dubbo.config.annotation.Service;
import org.springframework.stereotype.Component;

@Service
@Component
public class TicketServerImpl implements TicketServer {

    @Override
    public String getTicket(){
        return "ticket";
    }
}
```

4、运行，我们可以在dubbo-admin后台看到服务已经成功注册

#### 消费者

1、新建Spring maven项目，并在pom中添加dubbo和zookeeper依赖（同上）

2、在application.properties中配置

```
server.port=8002

dubbo.application.name=dubbo_consumer

dubbo.registry.address=zookeeper://127.0.0.1:2181
```

3、编写消费类

```java
package com.example.dubbo_consumer.server;

import com.example.dubbo_server.server.TicketServer;
import org.apache.dubbo.config.annotation.Reference;
import org.springframework.stereotype.Service;

@Service
public class UserServer {

    @Reference
    TicketServer ticketServer;

    public void buyTicket(){

        System.out.println(ticketServer.getTicket());
    }
}
```

注意，需要新建与提供者同样的TicketServer接口

4、编写测试实现

```java
@SpringBootTest
class DubboConsumerApplicationTests {

    @Autowired
    UserServer userServer;

    @Test
    void contextLoads() {
        userServer.buyTicket();
    }
}
```
