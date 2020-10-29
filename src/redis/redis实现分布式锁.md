# 一、什么是分布式锁？

要介绍分布式锁，首先要提到与分布式锁相对应的是线程锁、进程锁。

线程锁：主要用来给方法、代码块加锁。当某个方法或代码使用锁，在同一时刻仅有一个线程执行该方法或该代码段。**线程锁只在同一JVM中有效果**，因为线程锁的实现在根本上是依靠线程之间共享内存实现的，比如synchronized是共享对象头，显示锁Lock是共享某个变量（state）。

进程锁：为了控制同一操作系统中多个进程访问某个共享资源，因为进程具有独立性，各个进程无法访问其他进程的资源，因此无法通过synchronized等线程锁实现进程锁。

分布式锁：当多个进程不在同一个系统中，用分布式锁控制多个进程对资源的访问。

# 二、分布式锁的使用场景。

线程间并发问题和进程间并发问题都是可以通过分布式锁解决的，但是强烈不建议这样做！因为采用分布式锁解决这些小问题是非常消耗资源的！分布式锁应该用来解决分布式情况下的多进程并发问题才是最合适的。

有这样一个情境，线程A和线程B都共享某个变量X。

如果是单机情况下（单JVM），线程之间共享内存，只要使用线程锁就可以解决并发问题。

如果是分布式情况下（多JVM），线程A和线程B很可能不是在同一JVM中，这样线程锁就无法起到作用了，这时候就要用到分布式锁来解决。

# 三、分布式锁的实现（redis）

分布式锁实现的关键是在分布式的应用服务器外，搭建一个存储服务器，存储锁信息，这时候我们很容易就想到了Redis。首先我们要搭建一个Redis服务器，用Redis服务器来存储锁信息。
![1.jpg](https://tva1.sinaimg.cn/large/007S8ZIlly1gjmkfgz0fhj30x40j0jum.jpg)

在实现的时候要注意的几个关键点：
>1、锁信息必须是会过期超时的，不能让一个线程长期占有一个锁而导致死锁；
>2、同一时刻只能有一个线程获取到锁。

几个要用到的redis命令：

* `setnx(key, value)`：“set if not exits”，若该key-value不存在，则成功加入缓存并且返回1，否则返回0。
* `get(key)`：获得key对应的value值，若不存在则返回nil。
* `getset(key, value)`：先获取key对应的value值，若不存在则返回nil，然后将旧的value更新为新的value。
* `expire(key, seconds)`：设置key-value的有效期为seconds秒。
* `set（key，value，30，NX）`：setnx指令本身是不支持传入超时时间的，幸好Redis 2.6.12以上版本为set指令增加了可选参数。

# 四、redis锁解决并发问题
在工作中，我们有一个需求是打印面单，打印面单的时候需要向第三方获取物流单号，然后update到数据库，最后生成面单的链接返回给调用方。但是一旦并发（间隔十几毫秒），如下图

![2.jpg](https://tva1.sinaimg.cn/large/007S8ZIlly1gjmkft5z1cj309u0bvwgc.jpg)

两个请求并发进来的时候，检查到数据库都是没有物流单号的，所以都会往下执行。如果最后是请求1先落库，但却是请求2先返回的结果，这样就出现了数据库保存的和返回的结果不一致。因此为了解决这样并发造成的数据错乱，我们需要对请求加锁，保证并发情况下只有一个不同的请求处理。

```python
@try_exception
async def post(self):
    logging.info(f"请求打印发票的数据{self.get_body_args()}")
    kw = Schema(self.schema_order, ignore_extra_keys=True).validate(self.get_body_args())
    if not kw:
        self.resp({})
    seller_id = kw[0]['seller_id']
    seller_name = kw[0]['seller_name']
    seller_signature_uri = kw[0]['seller_signature_uri']
    request_body_base64 = base64.b64encode(str(self.get_body_args()).encode("utf-8"))[-20:]
    logging.info(f"request_body_base64:{request_body_base64}")
    async with Lock(redis_cli, 'print_invoice_body:' + str(request_body_base64), timeout=20):
        result = await MarketplacePDFService(seller_id, seller_name, seller_signature_uri).new_pdf_generate(kw)
    self.resp(result)

```

```python
# coding:utf-8
import asyncio
import logging
import time
import uuid

from tools_lib.host_info import WAREHOUSE_NAME
from .cedis import Cedis


class LockTimeout(Exception):
    pass


class TooFrequentException(Exception):
    pass

class Lock(object):
    def __init__(self, redis_client, key="", ttl=60, timeout=30, interval=0.03, is_wait=True, value=None):
        """
        :param key:
        :param ttl: expire time(锁最大过期时间) / s
        :param timeout: timeout(阻塞最大时间) / s
        :param interval: sleep interval(睡眠间隔) / s
        :return:
        Usage::

            with Lock('my_lock'):
                print "Critical section"
        """
        self.redis_client = redis_client
        self.key = f"redis:lock:{WAREHOUSE_NAME}:{key}"
        self.ttl = ttl
        self.timeout = timeout
        self.interval = interval
        # generate a unique UUID
        self.value = value or uuid.uuid1().hex
        self.is_wait = is_wait

    def is_acquired(self):
        return self.redis_client.get(self.key) == self.value and bool(self.value)

    def __enter__(self):
        self.acquire()

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.release()

    def acquire(self):
        timeout = self.timeout
        if timeout == 0:
            while True:
                if self.redis_client.set(self.key, self.value, ex=self.ttl, nx=True):
                    return True
                time.sleep(self.interval)
        else:
            while timeout >= 0:
                if self.redis_client.set(self.key, self.value, ex=self.ttl, nx=True):
                    return True
                timeout -= self.interval
                time.sleep(self.interval)
            raise LockTimeout("Timeout whilst waiting for lock")

    def release(self):
        if self.redis_client.get(self.key) == self.value:
            # only unlock myself lock
            return self.redis_client.delete(self.key)
        return 0

    async def __aenter__(self):
        res, wait_times = await self.async_acquire()
        return wait_times

    async def __aexit__(self, exc_type, exc, tb):
        self.release()

    async def async_acquire(self):
        timeout = self.timeout
        wait_times = 0  # 尝试获取锁的次数
        if timeout == 0:
            while True:
                wait_times += 1
                # redis操作理论上也需要改为异步，由于redis操作很快，暂时先用同步
                if self.redis_client.set(self.key, self.value, ex=self.ttl, nx=True):
                    return True, wait_times
                if not self.is_wait:  # 不等待锁
                    raise TooFrequentException('Operation is too frequent')
                await asyncio.sleep(self.interval)
        else:
            while timeout >= 0:
                wait_times += 1
                if self.redis_client.set(self.key, self.value, ex=self.ttl, nx=True):
                    return True, wait_times
                timeout -= self.interval
                if not self.is_wait:  # 不等待锁
                    raise TooFrequentException('Operation is too frequent')
                await asyncio.sleep(self.interval)
            raise LockTimeout("Timeout whilst waiting for lock")

```

## 参考
* [什么是分布式锁？](https://juejin.im/post/5b16148a518825136137c8db)
* [分布式锁的作用及实现（Redis）](https://blog.csdn.net/l_bestcoder/article/details/79336986)
* [浅谈分布式锁](http://www.linkedkeeper.com/1023.html)

