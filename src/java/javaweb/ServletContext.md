# Servlet

## ServletContext

servletcontext对象是用来保存共享数据，同一web应用中每一个servlet都可以共享。所以说servletcontext是个单例对象。

127.0.0.1:8080/hello->HelloServlet

```java
public class HelloServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/html");
        PrintWriter out = resp.getWriter();
        ServletContext content = this.getServletContext();
        content.setAttribute("username", "zhangsan");
        System.out.println(content);  // org.apache.catalina.core.ApplicationContextFacade@77c5a408
        out.println("This is my first java web page");
    }
}
```

127.0.0.1:8080/getcontext->GetContextServlet

```java
public class GetContextServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletContext context = this.getServletContext();
        System.out.println(context);  // org.apache.catalina.core.ApplicationContextFacade@77c5a408
        resp.setContentType("text/html");
        PrintWriter out = resp.getWriter();
        out.println(context.getAttribute("username"));
    }
}
```

重定向和转发的区别?

c

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gf9cseg88jj30fp09dmxa.jpg)


