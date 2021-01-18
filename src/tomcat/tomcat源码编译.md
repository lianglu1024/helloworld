# Tomcat源码编译

1.github上下载源码https://github.com/apache/tomcat，我们选择7.0.x版本。解压源码包

2.下载ant：https://archive.apache.org/dist/ant/binaries/apache-ant-1.10.8-bin.tar.gz

3.复制源码包中的build.properties.default变成build.properties，修改base.path，目的是下载的第三方依赖包放到源码包里

```
base.path=你的tomcat源码包/tomcat-build-libs
```

4.在tomcat-7.0.x目录下运行，估计会执行一段时间

```
ant ide-eclipse
```

5.使用idea导入源码包。File->New->Project From Existing Source，使用eclipse方式导入

6.工程绑定第三方依赖，我把ant下的lib包也一起复制到tomcat-build-libs目录下。idea的配置如下

![image-20210118114846801](https://tva1.sinaimg.cn/large/008eGmZEly1gmrpe6w7v6j30si0ibn2j.jpg)

6.进入org.apache.catalina.startup.Bootstrap，运行并可启动

## 乱码问题

1.org.apache.tomcat.util.res包下的StringManager

```java
    public String getString(final String key, final Object... args) {
        String value = getString(key);
        if (value == null) {
            value = key;
        }

        // 加入下面的代码
        try {
            value = new String(value.getBytes("ISO-8859-1"), "UTF-8");
        }catch(Exception e){
            e.printStackTrace();
        }
        //

        MessageFormat mf = new MessageFormat(value);
        mf.setLocale(locale);
        return mf.format(args, new StringBuffer(), null).toString();
    }
```

2.org.apache.jasper.compiler包下的Localizer

```java
    public static String getMessage(String errCode) {
        String errMsg = errCode;
        try {
            if (bundle != null) {
                errMsg = bundle.getString(errCode);
            }
        } catch (MissingResourceException e) {
        }

        //加入下面的代码
        try {
            errMsg = new String(errMsg.getBytes("ISO-8859-1"), "UTF-8");
        }catch(Exception e){
            e.printStackTrace();
        }
        //
      
        return errMsg;
    }
```

## 测试

1.新建空的maven工程，导入servlet-api依赖包。打包方式一定是war

```xml
<packaging>war</packaging>

<dependency>
  <groupId>javax.servlet</groupId>
  <artifactId>servlet-api</artifactId>
  <version>2.5</version>
</dependency>
```

2.新建com.zstu.servlet包，包下新建HelloServlet

```java
package com.zstu.servelt;

import javax.servlet.*;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class HelloServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("hello");
    }
}
```

3.在main目录下新建webapp->WEB_INF->web.xml

```xml
<!DOCTYPE web-app>

<web-app>
    <!--注册Servlet，helloservlet是随便取的，映射会用到-->
    <servlet>
        <servlet-name>helloservlet</servlet-name>
        <servlet-class>com.zstu.servelt.HelloServlet</servlet-class>
    </servlet>
    <!--一个Servlet对应一个Mapping映射-->
    <servlet-mapping>
        <servlet-name>helloservlet</servlet-name>
        <url-pattern>/hello</url-pattern>
    </servlet-mapping>
</web-app>
```

4.打包。选择HelloServlet下的Lifecycle->package打包。此刻target目录下会有一个HelloServlet-1.0-SNAPSHOT.war。把这个文件复制到tomcat源码包下的webapps下

5.启动tomcat并在浏览器上输入http://127.0.0.1:8080/HelloServelt-1.0-SNAPSHOT/hello，即可访问

![image-20210118132701510](https://tva1.sinaimg.cn/large/008eGmZEly1gmrs8etd34j30d70esgo6.jpg)

HelloServlet-1.0-SNAPSHOT.war解压之后的目录结构

![image-20210118132803311](https://tva1.sinaimg.cn/large/008eGmZEly1gmrs9hieaoj30d1065mxr.jpg)