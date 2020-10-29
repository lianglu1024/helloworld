# 第一个maven web项目

## 创建Maven webapp

![image-20200527194312559](https://tva1.sinaimg.cn/large/007S8ZIlly1gf78v5i208j30ob0k7afk.jpg)



![image-20200527194504551](https://tva1.sinaimg.cn/large/007S8ZIlly1gf78x18nduj30og03ljrl.jpg)



![image-20200527194643741](https://tva1.sinaimg.cn/large/007S8ZIlly1gf78yr338xj30oe08r0tv.jpg)

## 配置tomcat

![image-20200527194836671](https://tva1.sinaimg.cn/large/007S8ZIlly1gf790pjwpgj31870u0jyk.jpg)

![image-20200527195241787](https://tva1.sinaimg.cn/large/007S8ZIlly1gf794yj0ntj30tb0kh0xv.jpg)



![image-20200527195027474](https://tva1.sinaimg.cn/large/007S8ZIlly1gf792mqi9jj30te0jzn3i.jpg)

## 启动

![image-20200527195346744](https://tva1.sinaimg.cn/large/007S8ZIlly1gf7962qx3qj30ju09aacg.jpg)

## Servlet

### pom.xml下配置依赖包

```
<dependency> 
	<groupId>javax.servlet</groupId> 
	<artifactId>javax.servlet-api</artifactId> 
	<version>3.1.0</version> 
</dependency>
```

### 新建包

在main下新建java和resource目录，并把java mark Source root，resource mark成Resources root。新建包com.zstu.servlet，在下面新建HelloServlet.class

```java
package com.zstu.servlet;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

public class HelloServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/html");
        PrintWriter out = resp.getWriter();
        out.println("This is my first java web page");
    }
}
```

### 配置映射

web.xml

```xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
  <!--注册Servlet，helloservlet是随便取的，映射会用到-->
  <servlet>
    <servlet-name>helloservlet</servlet-name>
    <servlet-class>com.zstu.servlet.HelloServlet</servlet-class>
  </servlet>
  <!--一个Servlet对应一个Mapping映射-->
  <servlet-mapping>
    <servlet-name>helloservlet</servlet-name>
    <url-pattern>/hello</url-pattern>
  </servlet-mapping>
</web-app>
```

### 启动

输入http://127.0.0.1:8080/hello

## 工程结构

一般我们是新建一个空的maven父工程，然后在父工程里面新建多个子maven工程

### 新建空maven工程

### 新建子maven工程

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gf8dh7kr8hj30kn05at9o.jpg)

下面的过程跟上面新建web的maven工程一样

子工程新建完成后，父工程（abc_center）下的pom.xml多了

```xml
<modules>
  <module>abc_server</module>
</modules>
```

子工程（abc_server）下的pom.xml多了

```xml
<parent>
  <artifactId>abc_center</artifactId>
  <groupId>com.zstu</groupId>
  <version>1.0-SNAPSHOT</version>
</parent>
```

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gf8dopt53nj308j072t90.jpg)
