zookeeper是一种**分布式协调服务**，用于管理大型主机。在分布式环境中协调和管理服务是一个复杂的过程，zookeeper通过其简单的架构和API解决了这个问题。zookeeper允许开发人员专注于核心应用程序逻辑，而不必担心应用程序的分布式特性。

zookeeper的主要用途：
* 统一配置管理
* 统一命名服务
* 分布式锁
* 集群管理

# Zookeeper集群结构
![](https://tva1.sinaimg.cn/large/007S8ZIlly1gijcr7dox8j30go055mxg.jpg)

客户端可以通过服务端创建znode，删除znode，写znode,读znode，设置监视等等。

# zookeeper数据模型
zookeeper的节点是通过像树一样的结构来进行维护的，并且每一个节点通过路径来标示以及访问。除此之外，每一个节点还拥有自身的一些信息，包括：数据、数据长度、创建时间、修改时间等等。从这样一类既含有数据，又作为路径表标示的节点的特点中，可以看出，zookeeper的节点既可以被看做是一个文件，又可以被看做是一个目录，它同时具有二者的特点。与传统文件系统不同的是，zookeeper的文件目录树种保存的所有信息都在内存中。

![一个zk server上的节点树状图](https://tva1.sinaimg.cn/large/007S8ZIlly1gijcqx8fjhj30ca071q3h.jpg)

**注意：zookeeper节点不是zk broker，而是zk borker上维护的一种具有特定数据结构的对象**

# 安装zookeeper
安装zookeeper前必须要安装java
```
wget https://archive.apache.org/dist/zookeeper/zookeeper-3.4.10/zookeeper-3.4.10.tar.gz
tar -xf zookeeper-3.4.10.tar.gz
cd zookeeper-3.4.10/conf
mv zoo_sample.cfg zoo.cfg（zk默认是读取这个配置文件）
```
打开zoo.cfg
```
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=/Users/Liang/dataLog/zk/data_1
dataLogDir=/Users/Liang/dataLog/zk/log_1
# the port at which the clients will connect
clientPort=2181
```
如果要搭建集群的话还需要配置
```
server.A = B：C：D
			A：表示这个是第几号服务器，myid中的数字就是这个
			B：这个服务器的IP
			C：与集群中的leader交换信息的端口（默认2888）
			D：表示选举端口（默认是3888），集群中的leader挂了，需要一个端口用来进行选举，选出一个新的leader

例如
server.1=hadoop05:2888:3888
server.2=hadoop06:2888:3888
server.3=hadoop07:2888:3888
```
参数的含义：
* tickTime

tick翻译成中文的话就是滴答滴答的意思，连起来就是滴答滴答的时间，寓意心跳间隔，单位是毫秒，系统默认是2000毫秒，也就是间隔两秒心跳一次。tickTime的意义：客户端与服务器或者服务器与服务器之间维持心跳，也就是每个tickTime时间就会发送一次心跳。通过心跳不仅能够用来监听机器的工作状态，还可以通过心跳来控制Flower跟Leader的通信时间，默认情况下FL的会话时常是心跳间隔的两倍。

* initLimit

集群中的follower服务器(F)与leader服务器(L)之间初始连接时能容忍的最多心跳数（tickTime的数量）。

* syncLimit

集群中flower服务器（F）跟leader（L）服务器之间的请求和答应最多能容忍的心跳数。

* dataDir

存储快照文件snapshot的目录。默认情况下，事务日志也会存储在这里。建议同时配置参数dataLogDir, 事务日志的写性能直接影响zk性能。   

启动zk
```
./zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /Users/Liang/ops/zookeeper-3.4.10/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```
查看zk状态
```
$ ./zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /Users/Liang/ops/zookeeper-3.4.10/bin/../conf/zoo.cfg
Mode: standalone
```
关闭zk
```
$ ./zkServer.sh stop
```
### 客户端连接zk
```
$ ./zkCli.sh
Connecting to localhost:2181
2019-12-12 13:16:04,872 [myid:] - INFO  [main:Environment@100] - Client environment:zookeeper.version=3.4.10-39d3a4f269333c922ed3db283be479f9deacaa0f, built on 03/23/2017 10:13 GMT
2019-12-12 13:16:04,877 [myid:] - INFO  [main:Environment@100] - Client environment:host.name=192.168.131.177
2019-12-12 13:16:04,877 [myid:] - INFO  [main:Environment@100] - Client environment:java.version=1.8.0_111
2019-12-12 13:16:04,880 [myid:] - INFO  [main:Environment@100] - Client environment:java.vendor=Oracle Corporation
2019-12-12 13:16:04,880 [myid:] - INFO  [main:Environment@100] - Client environment:java.home=/Library/Java/JavaVirtualMachines/jdk1.8.0_111.jdk/Contents/Home/jre
...
```
**常用命令总结：**

| help             | 显示所有操作命令                                             |
| ---------------- | ------------------------------------------------------------ |
| ls path [watch]  | 使用 ls 命令来查看当前znode中所包含的内容                    |
| create [-s] [-e] | 普通创建。-s 含有序列，-e 临时（重启或者超时消失）           |
| get path [watch] | 获得节点的值。当使用watch的时候，如果另一个客户端修改节点值，watch的客户端会收到节点变更的信息，从而可以做相应的处理逻辑 |
| set              | 设置节点的具体值                                             |
| stat             | 查看节点状态                                                 |
| delete           | 删除节点                                                     |
| rmr              | 递归删除节点                                                 |



```
[zk: localhost:2181(CONNECTED) 2] create /sanguo "luoguanzhong"
Created /sanguo
[zk: localhost:2181(CONNECTED) 3] ls /
[zookeeper, sanguo]
[zk: localhost:2181(CONNECTED) 4] ls2 /
[zookeeper, sanguo]
cZxid = 0x0
ctime = Thu Jan 01 08:00:00 CST 1970
mZxid = 0x0
mtime = Thu Jan 01 08:00:00 CST 1970
pZxid = 0x7
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 2
[zk: localhost:2181(CONNECTED) 3] get /sanguo
lgz
cZxid = 0x7
ctime = Fri Dec 13 10:14:52 CST 2019
mZxid = 0xc
mtime = Fri Dec 13 11:16:36 CST 2019
pZxid = 0x7
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 3
numChildren = 0
[zk: localhost:2181(CONNECTED) 4] stat /sanguo
cZxid = 0x7
ctime = Fri Dec 13 10:14:52 CST 2019
mZxid = 0xc
mtime = Fri Dec 13 11:16:36 CST 2019
pZxid = 0x7
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 3
numChildren = 0
```
每个节点里都维护着cZxid，ctime，numChildren等节点信息

# 参考

 [Zookeeper简单介绍](https://www.cnblogs.com/sunddenly/p/4033574.html)
