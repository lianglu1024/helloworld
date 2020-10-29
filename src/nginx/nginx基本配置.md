### 什么是nginx？
nginx 是高性能的 HTTP 和反向代理的服务器，处理高并发能力是十分强大的，能经受高负 载的考验,有报告表明能支持高达 50,000 个并发连接数。

我们使用nginx的主要目的：
* 反向代理
其实客户端对代理是无感知的，因为客户端不需要任何配置就可以访问，我们只需要将请求发送到反向代理服务器，由反向代理服务器去选择目标服务器获取数据后，在返回给客户端，此时反向代理服务器和目标服务器对外就是一个服务器，暴露的是代理服务器地址，隐藏了真实服务器 IP 地址。
* 负载均衡
我们增加服务器的数量，然后将请求分发到各个服务器上，将原先请求集中到单个服务器上的情况改为将请求分发到多个服务器上，将负载分发到不同的服务器，也就是我们 所说的负载均衡
* 动静分离
![](https://upload-images.jianshu.io/upload_images/6853111-b0e601a4eeeaaee1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)
* 其他功能
白名单，访问频率

### 安装nginx
```
wget http://nginx.org/download/nginx-1.12.2.tar.gz
tar -xvf nginx-1.12.2.tar.gz
cd nginx-1.12.2
./configure
make && make install  # 编译并安装
```
安装完成会后，nginx被安装到`/usr/local/nginx`目录，在nginx有sbin启动脚本。启动nginx，我们可以在local:80看到Welcome to nginx!

常用命令
```
./nginx -s stop  # 关闭nginx
./nginx  # 启动nginx
./nginx -s reload  # 重加载（修改配置的情况下可以不stop）
```

#### 配置文件
当我们启动nginx的时候，nginx会默认加载`/usr/local/nginx/conf/nginx.conf`的配置文件。我们也可以指定加载自定义的配置文件。

```
# nginx.conf
worker_processes  1;

events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;

    keepalive_timeout  65;

    server {
        listen       80;
        server_name  localhost;

        location / {
            root   html;  # 入口文件的所在目录
            index  index.html index.htm;  # 默认入口文件名称
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
}
```
可见配置文件是包含三部分：

第一部分:全局块
从配置文件开始到 events 块之间的内容，主要会设置一些影响 nginx 服务器整体运行的配置指令，主要包括配 置运行 Nginx 服务器的用户(组)、允许生成的 worker process 数，进程 PID 存放路径、日志存放路径和类型以 及配置文件的引入等。

第二部分:events 块
events 块涉及的指令主要影响 Nginx 服务器与用户的网络连接，常用的设置包括是否开启对多 work process 下的网络连接进行序列化，是否允许同时接收多个网络连接，选取哪种事件驱动模型来处理连接请求，每个 word process 可以同时支持的最大连接数等。
上述例子就表示每个 work process 支持的最大连接数为 1024.

第三部分:http 块
这算是 Nginx 服务器配置中最频繁的部分，代理、缓存和日志定义等绝大多数功能和第三方模块的配置都在这里。 需要注意的是:http 块也可以包括 http 全局块、server 块。
* http 全局块
http 全局块配置的指令包括文件引入、MIME-TYPE 定义、日志自定义、连接超时时间、单链接请求数上限等。
* server 块
这块和虚拟主机有密切关系，虚拟主机从用户角度看，和一台独立的硬件主机是完全一样的，该技术的产生是为了 节省互联网服务器硬件成本。每个 http 块可以包括多个 server 块，而每个 server 块就相当于一个虚拟主机。 而每个 server 块也分为全局 server 块，以及可以同时包含多个 locaton 块。 
 `全局 server 块`：最常见的配置是本虚拟机主机的监听配置和本虚拟主机的名称或 IP 配置。
`location 块`：一个 server 块可以配置多个 location 块。这块的主要作用是基于 Nginx 服务器接收到的请求字符串(例如 server_name/uri-string)，对虚拟主机名称 (也可以是 IP 别名)之外的字符串(例如 前面的 /uri-string)进行匹配，对特定的请求进行处理。地址定向、数据缓 存和应答控制等功能，还有许多第三方模块的配置也在这里进行。

### 反向代理实例一
当我们访问127.0.0.1的时候，访问到的是http://www.baidu.com
只需如下配置
```
    server {
        listen       80;
        server_name  localhost;

        location / {
            proxy_pass  http://www.baidu.com;
            index  index.html index.htm;
        }
}
```
注意：`/`代表访问127.0.0.1:80下的任何资源（如127.0.0.1:80/xxx）都会访问到 http://www.baidu.com
### 反向代理实例二
根据访问的路径跳转到不同端口的服务中
 访问 http://127.0.0.1:9001/v1/ 直接跳转到 http://127.0.0.1:4001/
 访问 http://127.0.0.1:9001/v2/ 直接跳转到 http://127.0.0.1:4002/

新增server块
```
    server {
        listen       9001;
        server_name  localhost;

        location ~ /v1/ {
            proxy_pass http://127.0.0.1:4001;
        }

        location ~ /v2/ {
            proxy_pass http://127.0.0.1:4002;
        }
  
    }
```
**注意：当我们在浏览器上输入127.0.0.1:9001/v1/的时候，事实上nginx帮我们访问的是127.0.0.1:4001/v1/，并不是 http://127.0.0.1:4001**

在location指令和uri请求中间可以添加可选的修饰符，四种修饰符的含义分别如下


= 表示精确匹配。只有请求的url路径与后面的字符串完全相等时，才会命中。


~ 表示该规则是使用正则定义的，区分大小写。


~* 表示该规则是使用正则定义的，不区分大小写。


^~ 表示如果该符号后面的字符是最佳匹配，采用该规则，不再进行后续的查找。

### 负载均衡实例
当我们请求127.0.0.1的时候，请求会均衡到127.0.0.1:4005和127.0.0.1:4006上去。
```
    upstream myserver{
        server 127.0.0.1:4005;
        server 127.0.0.1:4006;
    }

    server {
        listen       80;
        server_name  localhost;

        location / {
            proxy_pass http://myserver;
        }
    }
```
而且 Nginx 提供了几种分配方式(策略):

**1、轮询(默认)**
每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器 down 掉，能自动剔除。

**2、weight**
weight 代表权,重默认为 1,权重越高被分配的客户端越多
```
upstream myserver{
server 192.168.5.21:80 weight =1; 
server 192.168.5.22:80 weight =1; 
}
```
**3、ip_hash**
每个请求按访问 ip 的 hash 结果分配，这样每个访客固定访问一个后端服务器，可以解决 session 的问题
```
upstream myserver{
ip_hash;
server 192.168.5.21:80; 
server 192.168.5.22:80; 
}
```
**4、fair**
按后端服务器的响应时间来分配请求，响应时间短的优先分配。
```
upstream myserver{
server 192.168.5.21:80; 
server 192.168.5.22:80; 
fair
}
```
### 动静分离实例
[https://klionsec.github.io/2017/12/21/nginx-static-dynamic/](https://klionsec.github.io/2017/12/21/nginx-static-dynamic/)

### nginx的config详解
[Nginx 基本配置详解](https://juejin.im/post/5aa7704c6fb9a028bb18a993)

[nginx配置文件详解中文版](https://www.jianshu.com/p/a7c86efe1987)
