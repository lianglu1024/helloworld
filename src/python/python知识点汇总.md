## with与async with
应用场景：
* 文件的读写
* 数据库的读写操作

**一. with**

上下文管理协议：当使用with语句时，解释器会自动调用 `__enter__`,`__exit__`

```python
class Sample:
    def __enter__(self):
        print('enter')  #进入资源
        return self
    def __exit__(self, exc_type, exc_val, exc_tb):
        print('exit')  #释放资源
    def do_something(self):
        print('do something')

with Sample() as sample:   
    sample.do_something()
```

输出

```
enter
do something
exit
```

进入with语句，调用`__enter__`；退出with语句，调用`__exit__`

**事实上sample并不是sample=Sample()，而是`__enter__`返回的对象，即如果`__enter__`没有return，sample将为None。**

exc_type：异常类型；exc_val：异常值；exc_tb：traceback。如果with语句中有异常发生，`__exit__`中会收集这些异常信息。

**二.async with**

async with的用法和with一样，只是内部使用`__aenter__`和`__aexit__`来定义上下文。这样我们就能在上下文中使用异步编程。

```
class Lock(object):
    def __init__(self, redis_client, key="", ttl=60, timeout=30, interval=0.03, is_wait=True):
        """
        :param key:
        :param ttl: expire time(锁最大过期时间) / s
        :param timeout: timeout(阻塞最大时间) / s
        :param interval: sleep interval(睡眠间隔) / s
        :return:
        Usage::

            with Lock('my_lock') as lock:
                pass
        """
        self.redis_client = redis_client
        self.key = f"redis:lock:{key}"
        self.ttl = ttl
        self.timeout = timeout
        self.interval = interval
        self.value = uuid.uuid1().hex
        self.is_wait = is_wait

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
## 描述符

