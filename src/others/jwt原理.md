# 传统session
互联网服务离不开用户认证。一般流程是下面这样。
```
1、用户向服务器发送用户名和密码。

2、服务器验证通过后，在当前对话（session）里面保存相关数据，比如用户角色、登录时间等等。

3、服务器向用户返回一个 session_id，写入用户的 Cookie。

4、用户随后的每一次请求，都会通过 Cookie，将 session_id 传回服务器。

5、服务器收到 session_id，找到前期保存的数据，由此得知用户的身份。
```

这种模式的问题在于，扩展性（scaling）不好。单机当然没有问题，如果是服务器集群，或者是跨域的服务导向架构，就要求 session 数据共享，每台服务器都能够读取 session。

举例来说，A 网站和 B 网站是同一家公司的关联服务。现在要求，用户只要在其中一个网站登录，再访问另一个网站就会自动登录，请问怎么实现？

一种解决方案是 session 数据持久化，写入数据库或别的持久层。各种服务收到请求后，都向持久层请求数据。这种方案的优点是架构清晰，缺点是工程量比较大。另外，持久层万一挂了，就会单点失败。

另一种方案是服务器索性不保存 session 数据了，所有数据都保存在客户端，每次请求都发回服务器。JWT 就是这种方案的一个代表。

# JSON Web Token（JWT）
使用jwt的最大优势就是服务端不需要存储session信息，减少了查询数据库比对过程。

那如何保证客户端传过来的token是有效的呢？我们先看一下服务端返回给客户端的token的形式是什么样的

> eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6IuW8gOWPkeiAhSIsImlzX2Nvb3BlcmF0aXZlX3N1cHBsaWVyIjp0cnVlLCJsb2dpbl9uYW1lIjoiZGV2ZWxvcGVyIiwiZXhwIjoxNTUyMzgzNTY0LCJ1c2VyX2lkIjoxLCJ1c2VyX3V1aWQiOiJTVVA2NzQ3NTk4NTciLCJlbWFpbCI6IjgyMzAyNjM2NUBxcS5jb20ifQ.VHbjWavmR13bhwovIBR5YE9LJMtl5Lyu89mB-M0yKUk

![](https://github.com/lianglu1024/helloworld/blob/master/%E8%A1%A5%E5%85%85%E7%9F%A5%E8%AF%86/pic/2.jpg)

可以看到token是由三部分组成，即

```
header+payload+signature
```

header和token是用base64加密的，可以反解码出来。signature是用密钥对header+payload进行加密的，所以服务端要看这个token是否有效，只要用自己的密钥对传过来的header+payload进行加密，将结果和传过来的signature比较即可。可以说，**jwt是用计算换存储的认证方式**。

## jwt工作流

![](https://github.com/lianglu1024/helloworld/blob/master/%E8%A1%A5%E5%85%85%E7%9F%A5%E8%AF%86/pic/1.jpg)

# 参考

* [JSON Web Token 入门教程](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html)
* [理解JWT（JSON Web Token）认证及python实践](https://segmentfault.com/a/1190000010312468)


