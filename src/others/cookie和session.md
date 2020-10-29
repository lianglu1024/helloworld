# cookie

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gfdrb75pqjj30hs0bo0tp.jpg)



# session

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gfdrbtoyzdj30hs0bwq3w.jpg)



Cookie是实现Session的**一种方式**，因为没有cookie, 也是可以实现Session的，可以将信息保存在URL里。

更一般的说，这是一个**目标和手段**的关系。

session本来就是一种抽象的概念，之所以抽象出来就是因为cookie有各种各样的限制

* cookie大小限制
* cookie明文保存在客户端不安全



你可以这样理解，既然cookie不安全，那么我用cookie保存一段加密字符串，只要能保证这段加密字符串可以被服务器唯一识别不就行了？这样session就被抽象出来了。这段加密字符串就是sessionId



概括来说，cookie是实现session的**一种方式**，因为没有cookie, 也是可以实现session的，可以将信息保存在URL里。更一般的说，这是一个**目标和手段**的关系。
