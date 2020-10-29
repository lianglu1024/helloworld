# Supervisor进程管理

supervisor就是用Python开发的一套通用的进程管理程序，能将一个普通的命令行进程变为后台daemon，并监控进程状态，异常退出时能**自动重启**。

**学习supervisor的主要重心就是如何配置和使用。**

**1.安装**

```
pip install supervisor
```

**2.安装完成后，导出默认的配置文件**

```
echo_supervisord_conf > default.conf
```

打开配置文件`default.conf`

```
[unix_http_server]
file=/tmp/supervisor.sock   ; UNIX socket 文件，supervisorctl 会使用，建议修改这个目录，如/var/supervisord/supervisor.sock
;chmod=0700                 ; socket 文件的 mode，默认是 0700
;chown=nobody:nogroup       ; socket 文件的 owner，格式： uid:gid
;username=user              ; default is no username (open server)
;password=123               ; default is no password (open server)

[inet_http_server]         ; HTTP 服务器，提供 web 管理界面
port=127.0.0.1:9001        ; Web 管理后台运行的 IP 和端口，如果开放到公网，需要注意安全性
username=user              ; 登录管理后台的用户名
password=123               ; 登录管理后台的密码

[supervisord]
logfile=/tmp/supervisord.log ; main log file; default $CWD/supervisord.log
logfile_maxbytes=50MB        ; 日志文件大小，超出会 rotate，默认 50MB
logfile_backups=10           ; 日志文件保留备份数量默认 10
loglevel=info                ; 日志级别，默认 info，其它: debug,warn,trace
pidfile=/tmp/supervisord.pid ; pid 文件(建议修改目录)
nodaemon=false               ; 是否在前台启动，默认是 false，即以 daemon 的方式启动
minfds=1024                  ; 可以打开的文件描述符的最小值，默认 1024
minprocs=200                 ; 可以打开的进程数的最小值，默认 200
;umask=022                   ; process file creation umask; default 022
;user=supervisord            ; setuid to this UNIX account at startup; recommended if root
;identifier=supervisor       ; supervisord identifier, default is 'supervisor'
;directory=/tmp              ; default is not to cd during start
;nocleanup=true              ; don't clean up tempfiles at start; default false
;childlogdir=/tmp            ; 'AUTO' child log dir, default $TEMP
;environment=KEY="value"     ; key value pairs to add to environment
;strip_ansi=false            ; strip ansi escape codes in logs; def. false

; The rpcinterface:supervisor section must remain in the config file for
; RPC (supervisorctl/web interface) to work.  Additional interfaces may be
; added by defining them in separate [rpcinterface:x] sections.

[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

; The supervisorctl section configures how supervisorctl will connect to
; supervisord.  configure it match the settings in either the unix_http_server
; or inet_http_server section.

[supervisorctl]
serverurl=unix:///tmp/supervisor.sock ; 通过 UNIX socket 连接 supervisord，路径与 unix_http_server 部分的 file 一致
;serverurl=http://127.0.0.1:9001 ; 通过 HTTP 的方式连接 supervisord
;username=chris              ; should be same as in [*_http_server] if set
;password=123                ; should be same as in [*_http_server] if set
;prompt=mysupervisor         ; cmd line prompt (default "supervisor")
;history_file=~/.sc_history  ; use readline history if available

[include]
files = relative/directory/*.conf ; 可以是 *.conf 或 *.ini，这里一般指定为自定义的config目录文件
```

**3.配置自己应用**

新建一个新的配置文件`myconf.conf`，以tornado的进程为例

```
[program:tornado]            ; [program：服务名]
command=/usr/bin/python -B /home/admin/apps/run_server.py --port=4002
directory=/home/admin/apps   ; 文件目录
autostart = false     ; 在 supervisord 启动的时候也自动启动
startsecs = 5        ; 启动 5 秒后没有异常退出，就当作已经正常启动了
autorestart = true   ; 程序异常退出后自动重启
startretries = 3     ; 启动失败自动重试次数，默认是 3
;user = leon          ; 用哪个用户启动
redirect_stderr = true  ; 把 stderr 重定向到 stdout，默认 false
stdout_logfile_maxbytes = 20MB  ; stdout 日志文件大小，默认 50MB
stdout_logfile_backups = 20     ; stdout 日志文件备份数
; stdout 日志文件，需要注意当指定目录不存在时无法正常启动，所以需要手动创建目录
stdout_logfile = /data/logs/xxxstdout.log
```

**4.启动supervisor并查看状态**

```
$ supervisord -c default.conf
$ supervisorctl status
```

Supervisor 有两个主要的组成部分：

* supervisord，运行 supervisor 时会启动一个进程 supervisord，它负责启动所管理的进程，并将所管理的进程作为自己的子进程来启动，而且可以在所管理的进程出现崩溃时自动重启。

* supervisorctl，是命令行管理工具，可以用来执行 stop、start、restart 等命令，来对这些子进程进行管理。

**5.可视化管理进程**

登录http://127.0.0.1:9001

![](https://ws4.sinaimg.cn/large/006tNc79ly1g26mkdkfkqj31860bodi6.jpg)

**6.杀死进程**

当我们收到kill掉运行着的tornado服务的时候，supervisor能够立刻重启tornado服务。



**参考：**

* [使用 Supervisor 管理服务器后台进程](https://zhuanlan.zhihu.com/p/36459081)

* [supervisor配置详解](https://www.cnblogs.com/ajianbeyourself/p/5534737.html)







