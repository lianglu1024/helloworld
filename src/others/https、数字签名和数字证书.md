https原理可以参考[一个故事讲完https](https://mp.weixin.qq.com/s?__biz=MzAxOTc0NzExNg==&mid=2665513779&idx=1&sn=a1de58690ad4f95111e013254a026ca2&scene=21#wechat_redirect)这篇文章

**大体流程如下：**
* client https请求server
* server保存了一套公钥和私钥，并返回公钥给client
* client随机生成一段字符串用作秘钥（key），并用公钥对秘钥加密传给server
* server用私钥解密出秘钥，之后client就可以和server使用秘钥加解密信息通信了

# 问题：
* 为啥不能直接用公钥加密消息，而是加密秘钥？

因为使用公钥私钥加解密消息消耗的时间长，使用对称加密更快

* 如何保证server端返回的数据没有被中间人篡改呢？

**数字签名**

先理解**消息摘要**，就是我们对消息进行摘要算法（MD5，SHA）生成的结果就是摘要。摘要一般是定长的字符串，消息中哪怕修改一点点，生成的摘要就会千差万别，所以一般用来验证数据是否篡改。

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gduicmzplsj30fa08374w.jpg)

我们（一般是服务端）拿着摘要用私钥加密，得到的就是签名（signature）

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gduidt4k60j30fa06t0sy.jpg)

为啥叫签名呢？一旦用私钥加密之后，那么这个加密的数据就是得到服务端的权威性认证，就像签名一样具有法定性认证。

server会把消息和签名一并发给client，client收到消息后会用公钥解开摘要，然后自己本地再对消息进行摘要算法，拿算出的摘要与解密出的摘要对比，如果相同，说明消息没有被篡改。**数字签名的作用就是用来验证消息有没有被篡改**

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gduigjhrlkj30fa08ggma.jpg)

* 如何保证client获取的公钥是server给的，而不是中间人发送的虚假公钥？

**数字证书**

数字证书一般由数字证书认证机构（Certificate Authority）颁发，证书里面包含了真实服务器的公钥和网站的一些其他信息，数字认证机构用自己的私钥加密后发给浏览器，浏览器使用数字证书机构的公钥解密后得到真实服务器的公钥。这个过程是建立在被大家所认可的证书机构之上得到的公钥，所以这是一个安全的方式。

## 进阶版流程
![](https://tva1.sinaimg.cn/large/007S8ZIlly1gdunihhpf5j30hs0k2jtc.jpg)
