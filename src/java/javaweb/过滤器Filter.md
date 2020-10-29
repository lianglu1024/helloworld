# Filter

filter的作用是对请求的进入和出去做统一处理，而不需要对每一个servlet编写单独的逻辑处理。

编写自定义的filter需要实现Filter接口的三个方法：

* `public void init(FilterConfig filterConfig)`：web容器启动时加载
* `public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain)`：具体要过滤的内容
* `public void destroy()`：web容器关闭时执行

## 新建filter

在包com.zstu.filter下新建`CharacterEncodingFilter`

```java
package com.zstu.filter;

import javax.servlet.*;
import java.io.IOException;

public class CharacterEncodingFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("filter初始化...");
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        servletResponse.setCharacterEncoding("utf-8");
        servletRequest.setCharacterEncoding("utf-8");
        System.out.println("servlet执行前...");
        filterChain.doFilter(servletRequest, servletResponse);
        System.out.println("servlet执行后...");
    }

    @Override
    public void destroy() {
        System.out.println("filter销毁...");
    }
}
```

注意：filterChain.doFilter的作用就是去执行对应的servlet

## 配置xml

在web.xml下配置filter映射

```xml
<filter>
  <filter-name>characterencodingfilter</filter-name>
  <filter-class>com.zstu.filter.CharacterEncodingFilter</filter-class>
</filter>
<filter-mapping>
  <filter-name>characterencodingfilter</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
```

`/*`表示对每个请求都做过滤操作



## 执行流程

* tomcat启动时，会去执行所有filter的`init`方法
* 当请求进来的时候，会根据xml配置的`url-pattern`来决定哪些需要走filter
* 执行filter的doFilter方法。如果有多个filter，会依次执行，`chain.doFilter`方法具体执行流程可以看下面

```
......
CharacterEncodingFilter初始化...
ExtraFilter初始化...
[2020-06-02 02:14:53,807] Artifact abc_server:war: Artifact is deployed successfully
[2020-06-02 02:14:53,807] Artifact abc_server:war: Deploy took 795 milliseconds
02-Jun-2020 14:15:02.943 信息 [localhost-startStop-1] org.apache.catalina.startup.HostConfig.deployDirectory 把web 应用程序部署到目录 [/Users/Liang/opt/apache-tomcat-8.5.51/webapps/manager]
02-Jun-2020 14:15:02.985 信息 [localhost-startStop-1] org.apache.catalina.startup.HostConfig.deployDirectory Deployment of web application directory [/Users/Liang/opt/apache-tomcat-8.5.51/webapps/manager] has finished in [42] ms
CharacterEncodingFilter-servlet执行前...
ExtraFilter-servlet1执行前...
org.apache.catalina.core.ApplicationContextFacade@49d0c80
ExtraFilter-servlet1执行后...
CharacterEncodingFilter-servlet执行后...
```

