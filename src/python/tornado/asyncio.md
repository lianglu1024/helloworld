我们希望我们的异步IO执行是怎么的呢？假设定义三个异步任务

```python
def async_worker1():
    pass
		
def async_worker2():
    pass
    
def async_worker3():
    pass
    
tasks = [async_worker1, async_worker2, async_worker3]
do(tasks)
```

将三个异步任务丢到一个tasks里去执行。当执行async_worker1的时候，如果里面有IO请求或者定时任务，async_worker1需要暂停，将主动权交给调用方去执行另外的任务。只有当注册事件发生或者定时时间到来时，才会调用回调函数去执行相应的操作。

要设计这样的异步执行流程，需要考虑以下几点：
* 异步任务里肯定会有yield，否则异步任务没法暂停执行
* 必须要有一个事件循环不断的去检查事件是否发生和定时任务
* 一定有人执行send函数，让异步任务从暂停位置继续执行下去

## asyncio四大模块
### Future
* `_callbacks`：回调函数集合
* `_schedule_callbacks`：将`_callbacks`里的回调函数丢进`_ready`准备执行
* `result()`：返回future代表的结果
* `add_done_callback(fn)`：当future完成之后去执行回调fn（其实是丢到`_ready`）；如果没有完成，先丢到`_callbacks`里
* `set_result(result)`：将future标为执行完成，然后设置result的值，最后调用`_schedule_callbacks`

想要执行回调函数绑定在future上的回调，需要由调用方调用set_result方法

Future的使用：

```python
import asyncio

def mark_done(future, result):
    print(f'setting future result to {result}')
    future.set_result(result)


event_loop = asyncio.get_event_loop()
all_done = asyncio.Future()

print('scheduling make_done')
event_loop.call_soon(mark_done, all_done, 'the result')
print('entering event loop')
result = event_loop.run_until_complete(all_done)
print(f'returned result: {result}')
print('closing event loop')
event_loop.close()
print(f'future result: {all_done.result()}')
```

当调用 `set_result` 方法后，`Future` 对象的状态会被修改为 done, 同时 `Future`实例也会保存设置的结果值，供随后使用

```
scheduling make_done
entering event loop
setting future result to the result
returned result: the result
closing event loop
future result: the result
```

`Future` 对象也可以同 `await` 关键字一起使用:

```python
import asyncio

def mark_done(future, result):
    print(f'setting future result to {result}')
    future.set_result(result)

async def main(loop):
    all_done = asyncio.Future()
    print('scheduling mark_done')
    loop.call_soon(mark_done, all_done, 'the result')
    result = await all_done
    print(f'returned result: {result}')


event_loop = asyncio.get_event_loop()
event_loop.run_until_complete(main(event_loop))
event_loop.close()
```

`await` 会返回 `Future` 的结果：

```
scheduling mark_done
setting future result to 'the result'
returned result: the result
```

`Future` 在完成的时候可以执行一些回调函数，回调函数按注册时的顺序进行调用:

```python
import asyncio
import functools


def callback(future, n):
    print(f'{n}: future done: {future.result()}')


async def register_callbacks(all_done):
    print('registering callbacks on future')
    all_done.add_done_callback(functools.partial(callback, n=1))
    all_done.add_done_callback(functools.partial(callback, n=2))


async def main(all_done):
    await register_callbacks(all_done)
    print('setting result of future')
    all_done.set_result('the result')


event_loop = asyncio.get_event_loop()

all_done = asyncio.Future()
event_loop.run_until_complete(main(all_done))
event_loop.close()
```

回调函数的第一个参数是 `Future` 实例，要传递其他参数可以使用 `functools.partial()` 来实现。

```
registering callbacks on future
setting result of future
1: future done: the result
2: future done: the result
```
### Task

Task是Future的子类，拥有Future的属性和方法

* `_step`：运行协程（即send(None)）。将`task._wakeup`添加到协程yield的future的`_callbacks`里。记住，只有调用方调用future.set_result的时候，即future完成时才会执行`_callbacks`里的回调函数；由此可见只有异步请求返回结果之后才会去执行接下去未完成的任务
* `_wakeup`：执行`_step`函数，其实就是从协程暂停继续往下执行

任何协程在运行之前，会被封装成task对象。在封装成task的时候，`_step`会被丢进`_ready`等待执行


### Eventloop

* `_ready`：所有的回调函数执行，都是从`_ready`里获取的
* `_scheduled`：里面存放的是延迟的回调函数，不过最终还是丢到`_ready`去执行
* `call_soon`：将回调函数丢进`_ready`里
* `call_later`：将回调函数丢进`_scheduled`里
* `run_until_complete`：开启无限循环，执行`_ready`里的任务

## 参考
* [asyncio源码分析之基本执行流程](https://www.jianshu.com/p/cad64cd739e8)
