# 快速入门

## 硬核开发

1. 下载tomcat（apache-tomcat-8.5.51）
2. 在webapps目录下新建myapp文件夹（这就是web项目的名称）
3. 在myapp文件夹下新建hello.html和WEB-INF文件夹
4. 如果开启tomcat，输入127.0.0.0:8080/myapp/hello.html即可访问静态资源文件
5. 如果访问动态资源文件，在WEB-INF下新建classes和lib文件夹以及web.xml

新建HelloServlet.java

```java
import javax.servlet.*;

import java.io.IOException;

public class HelloServlet implements Servlet {
    @Override
    public void init(ServletConfig servletConfig) throws ServletException {
        System.out.println("init方法调用");
    }

    @Override
    public ServletConfig getServletConfig() {
        System.out.println("getServletConfig方法调用");
        return null;
    }

    @Override
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
        System.out.println("service方法调用");
    }

    @Override
    public String getServletInfo() {
        System.out.println("getServletInfo方法调用");
        return null;
    }

    @Override
    public void destroy() {
        System.out.println("destroy方法调用");
    }
}
```

编译文件（从tomcat的lib下拷贝servlet-api.jar到HelloServlet.java同级目录下）

```
javac -classpath servlet-api.jar HelloServlet.java
```

将生成的 HelloServlet.class文件拷贝到classes目录下

编辑web.xml

```xml
<!DOCTYPE web-app>

<web-app>
  <!--注册Servlet，helloservlet是随便取的，映射会用到-->
  <servlet>
    <servlet-name>helloservlet</servlet-name>
    <servlet-class>HelloServlet</servlet-class>
  </servlet>
  <!--一个Servlet对应一个Mapping映射-->
  <servlet-mapping>
    <servlet-name>helloservlet</servlet-name>
    <url-pattern>/hello</url-pattern>
  </servlet-mapping>
</web-app>
```

最终的目录结构如下：

![image-20200922111010370](https://tva1.sinaimg.cn/large/007S8ZIlly1giz95nht4mj30fo06r3yz.jpg)

6. 重启tomcat，输入http://127.0.0.1:8080/myapp/hello，观察日志

`myapp`是放在webapps下的项目名，`hello`是web.xml里定义的url-pattern，它会去找匹配的Servlet实现类

## IDEA

1. 新建一个java web项目servlet01

![image-20200922111539195](https://tva1.sinaimg.cn/large/007S8ZIlly1giz9bckb6fj30mt0k1jto.jpg)

2. 新建一个包com.zstu.servlet，在该包下创建一个MyServelt，内容同上

3. 点击Edit Configuration，来配置Tomcat

![image-20200922114350692](https://tva1.sinaimg.cn/large/007S8ZIlly1giza4owtpjj30tg0g440g.jpg)

![image-20200922114307411](https://tva1.sinaimg.cn/large/007S8ZIlly1giza3xnl6lj30pl09hq3m.jpg)

4. 运行tomcat01，访问http://127.0.0.1:8080/hello

## 热部署和war包

上面我们部署成exploded，就是开发模式。当我们修该代码之后，直接运行tomcat可以看到运行结果，这个方便于开发，称为热部署。

生产部署的时候，我们要将项目生成war包，然后将war包拷贝到webapps下。tomcat启动时会解压.war包文件，访问方式就跟最上面的硬核开发一样。

如何将项目打成war包呢？

![image-20200922130421751](https://tva1.sinaimg.cn/large/007S8ZIlly1gizcggkswaj30wk0npwmg.jpg)

Build->Build Artifact->xxx.war->build；在out.artifact文件夹下生产了xxx.war，我们只需将这个war包拷贝到webapps即可运行tomcat，tomcat在启动的时候会自动解压war包。


## Servlet

Servlet是一套接口规范，定义了web应用需要执行的方法：

![image-20200922102305922](https://tva1.sinaimg.cn/large/007S8ZIlly1giz7sphrynj30ao0390ss.jpg)

实现Servlet接口的常用类有GenericServlet，HttpServlet，其中GenericServlet是对Servlet简单的实现，不怎么使用，HttpServlet继承GenericServlet，封装了Http的常用操作。

```java
package com.zstu.servlet;

import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;


public class MyServlet extends HttpServlet {
    @Override
    public void doGet(HttpServletRequest req, HttpServletResponse resp){
        System.out.println("this is get");
    }
}
```

当请求进来之后，tomcat会调用MyServlet对象的service方法，HttpServlet对象的service方法里面根据请求的方式来选择调用doGet还是doPost还是其他，我们只需重写对应的方法即可。

### web.xml

```xml
<!DOCTYPE web-app>
<web-app>
  <!--注册Servlet，helloservlet是随便取的，映射会用到-->
  <servlet>
    <servlet-name>helloservlet</servlet-name>
    <servlet-class>HelloServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
  </servlet>
  <!--一个Servlet对应一个Mapping映射-->
  <servlet-mapping>
    <servlet-name>helloservlet</servlet-name>
    <url-pattern>/hello</url-pattern>
  </servlet-mapping>
</web-app>

url-pattern定义匹配规则：
精确匹配  /具体的名称  只有url路径是具体的名称时候才会触发Servlet
后缀匹配  *.xxx      只有以xxx结尾的就匹配触发Servlet
通配匹配  /*         匹配所有的请求，包含服务器的所有资源
通配匹配  /          匹配所有请求，包含服务器的所有资源，不包括jsp

load-on-startup
1.元素标记容器是否应该在web应用程序启动的时候就加载这个servlet
2.它的值必须是一个整数，表示servlet被加载的先后顺序
3.如果该值为负数或者没有设置，则容器当做Servlet被请求时再加载
4.如果值为正数或者0时，表示容器在应用启动时就加载并初始化这个servlet，值越小，servlet的优先级越高，就越被先加载。值相同时，容器就会自己选择顺序加载
```

### 注解方式

```java
package com.zstu.servlet;

import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@WebServlet(value = {"/bs"}, loadOnStartup = -1)
public class GenServlet extends HttpServlet {
    @Override
    public void doGet(HttpServletRequest req, HttpServletResponse resp){
        System.out.println("this is get");
    }
}
```

## Servlet生命周期

实例化：当请求进来之后创建Servlet对象（如果配置了load-on-start-up，是在tomcat启动时）。**该Servlet对象只创建一次**

初始化：init()，**该方法只调用一次**

服务：service()

销毁：destroy()，Servlet容器停止会调用

![Servlet生命周期- 三只小菜鸟- 博客园](https://tva1.sinaimg.cn/large/007S8ZIlly1gizidwymx8j30tl0gnn2i.jpg)

```java
package com.zstu.servlet;

import javax.servlet.ServletConfig;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;


@WebServlet(value = {"/bs"}, loadOnStartup = -1)
public class GenServlet extends HttpServlet {
    public GenServlet(){
        System.out.println("Servlet实例化");
    };

    @Override
    public void init(ServletConfig var1){
        System.out.println("Servlet初始化");
    }

    @Override
    public void doGet(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        System.out.println("service执行");
        PrintWriter writer = resp.getWriter();
        writer.write("service执行");
        writer.flush();
        writer.close();
    }

    @Override
    public void destroy(){
        System.out.println("销毁");
    }
}
```

访问：http://localhost:8080/bs

```
实例化
初始化
service执行
```

关闭tomcat，会执行destory方法

### Servlet多线程

由于运行阶段只有一个XXXServlet对象，在访问公共变量的时候存在竞争关系。

解决多线程并发问题：

* synchronized
* 尽可能使用局部变量

## ServletContext对象

全局对象，当web服务器启动时，会为每一个web应用程序创建一块共享的存储区域（ServletContext）。ServletContext在web服务器启动时创建，服务器关闭时销毁。

获取ServletContext对象：

* GenericSevlet提供了getSevletContext方法。this.getSevletContext();
* HttpServletRequest提供了getSevletContext方法

ServletContext作用：

* 获取路径
* 全局容器

```java
    @Override
    public void doGet(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        System.out.println("service执行");
//        ServletContext servletContext = this.getServletContext();
        ServletContext servletContext = req.getServletContext();

        System.out.println(servletContext.getRealPath("/"));  // 项目路径

        System.out.println(req.getContextPath());  // 上下文路径

        servletContext.setAttribute("name", "lisi");

        //供其他servlet对象调用使用
        System.out.println(servletContext.getAttribute("name")); //lisi
        PrintWriter writer = resp.getWriter();
        writer.write("service执行");
        writer.flush();
        writer.close();
    }
```

## Filter过滤器

执行时间在Servlet之前，客户端发送请求时，会先经过Filter，再达到目标Servlet；响应时会根据执行流程再次反向执行Filter。过滤器的作用：解决多个Servlet共性代码的冗余问题（例如登录验证，字符编码）。

实现过程：

* 编写java类实现Filter接口
* 在doFilter方法中编写拦截逻辑
* 设置拦截路径

```java
package com.zstu.filter;

import javax.servlet.*;
import javax.servlet.annotation.WebFilter;
import java.io.IOException;

@WebFilter(value = {"/bs"})
public class MyFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("过滤器init");
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("filter begin");
        //将请求向下传递
        filterChain.doFilter(servletRequest,servletResponse);
        System.out.println("filter end");
    }

    @Override
    public void destroy() {
        System.out.println("过滤器destroy");
    }
}
```

![image-20200922184618650](https://tva1.sinaimg.cn/large/007S8ZIlly1gizmc949e5j30i608274o.jpg)

过滤器链：客户端对服务器请求之后，服务器调用Servlet之前会执行一组过滤器（多个过滤器），那么这组过滤器称为过滤器组。

```java
package com.zstu.filter;

import javax.servlet.*;
import javax.servlet.annotation.WebFilter;
import java.io.IOException;

@WebFilter(value = {"/bs"})
public class MyFilter2 implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("过滤器2init");
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("filter2 begin");
        filterChain.doFilter(servletRequest,servletResponse);
        System.out.println("filter2 end");
    }

    @Override
    public void destroy() {
        System.out.println("过滤器2destroy");
    }
}
```

执行：

```
过滤器init
过滤器2init
[2020-09-22 06:48:02,980] Artifact servlet01:war exploded: Artifact is deployed successfully
[2020-09-22 06:48:02,980] Artifact servlet01:war exploded: Deploy took 570 milliseconds
servlet实例化
servlet初始化
filter begin
filter2 begin
service执行
filter2 end
filter end
# /Users/Liang/opt/apache-tomcat-8.5.51/bin/catalina.sh stop
servlet销毁
过滤器destroy
过滤器2destroy
```

过滤器优先级：

* 如果为注解的话，是按照类全名称的字符串顺序
* 如果为web.xml，按照filter-mapping注册顺序，从上到下
* web.xm配置高于注解方式

补充：web.xml配置Filter

```xml
<filter>
  <filter-name>filter01</filter-name>
  <filter-class>com.zstu.filter.MyFilter</filter-class>
</filter>
<filter-mapping>
  <filter-name>filter01</filter-name>
  <url-pattern>/bs</url-pattern>
</filter-mapping>
```
