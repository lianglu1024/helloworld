# SSH

简单说，SSH是一种网络协议，用于计算机之间的加密登录。如果一个用户从本地计算机，使用SSH协议登录另一台远程计算机，我们就可以认为，这种登录是安全的，即使被中途截获，密码也不会泄露。

那么如何对数据进行加密呢？通常有对称加密和非对称加密两种，其中非对称加密使用范围更广。

## 对称加密

所谓对称加密，指加密解密使用同一套秘钥。

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gdtk07idbxj30bl06ijrs.jpg)

对称加密的加密强度高，很难破解。但是在实际应用过程中不得不面临一个棘手的问题：**如何安全的保存密钥呢？**尤其是考虑到数量庞大的Client端，很难保证密钥不被泄露。一旦一个Client端的密钥被窃据，那么整个系统的安全性也就不复存在。为了解决这个问题，**非对称加密**应运而生。非对称加密有两个密钥：**“公钥”**和**“私钥”**。

## 非对称加密

非对称加密有两个密钥：**“公钥”**和**“私钥”**。ssh使用非对称加密的过程如下：

* 远程Server收到Client端用户的登录请求，Server把自己的公钥发给用户。

* Client使用这个公钥，将密码进行加密。

* Client将加密的密码发送给Server端。

* 远程Server用自己的私钥，解密登录密码，然后验证其合法性。

* 若验证结果，给Client相应的响应。

私钥是Server端独有，这就保证了Client的登录信息即使在网络传输过程中被窃据，也没有私钥进行解密，保证了数据的安全性，这充分利用了非对称加密的特性。

如何生成公钥和私钥呢？最著名的就是RSA算法，可以参考阮一峰的[RSA算法原理1](https://www.ruanyifeng.com/blog/2013/06/rsa_algorithm_part_one.html)和[RSA算法原理2](http://www.ruanyifeng.com/blog/2013/07/rsa_algorithm_part_two.html)

### 中间人攻击

上述流程会有一个问题：Client端如何保证接受到的公钥就是目标Server端的？如果一个攻击者中途拦截Client的登录请求，向其发送自己的公钥，Client端用攻击者的公钥进行数据加密。攻击者接收到加密信息后再用自己的私钥进行解密，不就窃取了Client的登录信息了吗？这就是所谓的中间人攻击

在https中可以通过CA来进行公证，可是SSH的publish key和private key都是自己生成的，没法公证。**只能通过Client端自己对公钥进行确认**。通常在第一次登录的时候，系统会出现下面提示信息

```
$ ssh root@10.211.55.3
The authenticity of host '10.211.55.3 (10.211.55.3)' can't be established.
ECDSA key fingerprint is SHA256:KHWdPYoQFkskwn87uMrNeZq8uhyaQtQfnP5Isxmf8io.
Are you sure you want to continue connecting (yes/no)? yes

```

上面的信息说的是：无法确认主机10.211.55.3 (10.211.55.3)的真实性，不过知道它的公钥指纹，是否继续连接？

如果输入yes后，会出现下面信息：

```
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '10.211.55.3' (ECDSA) to the list of known hosts.
```
我们可以在~/.ssh/known_hosts文件下看到10.211.55.3被追加进去，下次远程登录的时候，就不会出现上面的警告和确认信息。

## 免密码登录
每次远程登录的时候都要输入密码，那如何免密码登录呢？好在SSH还提供了公钥登录，可以省去输入密码的步骤。

所谓"公钥登录"，原理很简单，就是用户将自己的公钥储存在远程主机上。登录的时候，远程主机会向用户发送一段随机字符串，用户用自己的私钥加密后，再发回来。远程主机用事先储存的公钥进行解密，如果成功，就证明用户是可信的，直接允许登录shell，不再要求密码。
> 注：用公钥加密的数据只有对应的私钥可以解密；用私钥加密的数据只有对应的公钥可以解密

这种方法要求用户必须提供自己的公钥。如果没有现成的，可以直接用ssh-keygen生成一个：

```
$ ssh-keygen
```

## 参考
* [SSH原理与运用（一）：远程登录](https://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html)

