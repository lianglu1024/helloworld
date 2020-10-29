# 协程
## 生成器
如果一个函数中出现了关键字yield，那么这个函数就是生成器。yield既有产出，也有让步的意思，很符合生成器的特性。
```python
In [1]: def simple_coroutine():
   ...:     print('-> coroutine started')
   ...:     x = yield 20
   ...:     print('-> coroutine received:', x)
   ...:     return 30
   ...:

In [2]: my_coro = simple_coroutine()

In [3]: y = next(my_coro)
-> coroutine started

In [4]: y
Out[4]: 20

In [5]: my_coro.send(50)
-> coroutine received: 50
-------------------------------------------------------------------
StopIteration                     Traceback (most recent call last)
<ipython-input-5-c7bbd2cbcd81> in <module>
----> 1 my_coro.send(50)

StopIteration: 30
```
* 使用send(value)的时候，生成器必须是预激状态。要想生成器处于预激状态，可以使用next或者send(None)
* 当执行next的时候，生成器会将yield后的值产生给调用方，并暂停生成器，程序的主动权交给调用方
* send(value)的时候，生成器从暂停位置激活，并把value传给yield等号左侧的变量，继续执行后面的语句
* 通常无法获得生成器的return值，除非捕获异常从里面取值或者用yield from表达式
### 预激生成器
我们知道，使用生成器之前，必须要激活生成器，因此我们可以用一个装饰器来装饰生成器，这样子就省去了手动激活的操作了
```python
from functools import wraps

def coroutine(func):
    @wraps(func)
    def primer(*args, **kwargs):
        gen = func(*args, **kwargs)
        next(gen)
        return gen
    return primer

@coroutine
def simple_coroutine():
    while True:
        x = yield
        print(x)

if __name__ == '__main__':
    my_cor = simple_coroutine()
    my_cor.send(10)
    my_cor.send(20)
```
### yield from
yield from作用：
* 替代产出值的嵌套for循环
* 获得子生成器的return值
* 打开双向通道，把最外层的调用方与最内层的子生成器连接起来，这样两者直接发送和产出值
#### 简单理解
`yield from 可迭代对象(子生成器)`，既然yield from是一个表达式，那么就可以赋值给变量，该变量就是子生成器的return值
```
yield from 可迭代对象
等价于
for i in 可迭代对象:
    yield i
```
例子
```
In [6]: def f():
   ...:     yield from "abc"
   ...:

In [7]: m = f()

In [8]: next(m)
Out[8]: 'a'

In [9]: next(m)
Out[9]: 'b'

In [10]: next(m)
Out[10]: 'c'
```
#### 高级用法
![](https://tva1.sinaimg.cn/large/007S8ZIlly1gebn9ag3e8j31680eagx3.jpg)
```python
from collections import namedtuple

Result = namedtuple('Result', 'count average')

# 子生成器
def averager():
    total = 0.0
    count = 0
    average = None
    while True:
        term = yield
        if term is None:
            break
        total += term
        count += 1
        average = total / count
    return Result(count, average)

# 委托生成器
def grouper(results, key):
    while True:
        results[key] = yield from averager()

# 客户端代码，即调用方
def main(data):
    results = {}
    for key, values in data.items():
        group = grouper(results, key)
        next(group)
        for value in values:
            group.send(value)
        group.send(None)
    report(results)

# 输出报告
def report(results):
    for key, result in sorted(results.items()):
        group, unit = key.split(';')
        print(f"{result.count}{group} averaging {result.average}{unit}")

data = {
    'girls;kg': [40.9, 38.5],
    'girls;m': [1.6, 1.51],
    'boys;kg': [39.0, 40.8],
    'boys;m': [1.38, 1.5]
}

if __name__ == '__main__':
    main(data)
```
