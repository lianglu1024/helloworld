## Python内置的数据结构
```python
DataStructure = {'Collections': {'Map': [('dict', 'OrderDict', 'defaultdict'),
                                         ('chainmap', 'types.MappingProxyType')],
                                 'Set': [('set', 'frozenset'), {'multiset':'collection.Counter'}]},
                                 'Sequence': {'Basic': ['list', 'tuple', 'iterator']},
                                              'Algorithm': {'Priority': ['heapq',
                                                                         'queue.PriorityQueue'],
                                                            'Queue': ['queue.Queue',
                                                                      'multiprocessing.Queue'],
                                                            'Stack': ['collection.deque',
                                                                      'queue.LifeQueue']},
                 'text_sequence': ['str', 'byte', 'bytearray']}
```



## 1.堆heap

`heapq`模块提供了堆的实现，默认是最小堆。`queue`模块中的优先队列`PriorityQueue`就是基于`heapq`模块实现的

```python
import heapq

heap = [] # 定义一个数组，元素是按照完全二叉树的结构存放在数组中的
heapq.heappush(heap,2)
heapq.heappush(heap,4)
heapq.heappush(heap,3)
print(heap)  # [2, 4, 3]
print(heapq.heappop(heap)) # 默认是最小堆，弹出最小元素
mylist = [4,2,6]
heapq.heapify(mylist) # 将数组转变成最小堆
print(mylist) # [2, 4, 6]
```

如何将最小堆转成最大堆呢？其实只用对元素取负，按照最小堆的方式存储，取出来的值只要再取负，就是最大值了。即push(e)改为push(-e)，pop(e)为-pop(e)

### 优先队列

```python
from queue import PriorityQueue

pq = PriorityQueue()

pq.put(5)
pq.put(10)
pq.put(12)
pq.put(3)
print(pq.qsize())  # 4
print(pq.get())   # 3
print(pq.get())  # 5
print(pq.qsize()) # 2
```



## 2.双端队列deque

双端队列（deque，全名double-endedqueue）是一种具有队列和栈的性质的数据结构。双端队列中的元素可以从两端弹出，其限定插入和删除操作在表的两端进行。

Python中双端队列的操作函数；

1. 先调用模块 ：`from collections import deque` #首先从collections 模块中导入deque类
2. 定义一个双端队列和操作

```
- A=deque() #创建一个空的双队列
- A.append(n) #从右边像队列中增加元素 ，n表示增加的元素
- A.appendleft(n) #从左边像队列中增加元素，n表示增加的元素
- A.clear() #清空队列
- A.count(n) #在队列中统计元素的个数，n表示统计的元素
- A.extend(n) #从右边扩展队列，n表示扩展的队列
- A.extendleft(n) #从左边扩展队列，n表示扩展的队列
- A.pop() #从队列的右边删除元素，并且返回删除值
- A.popleft() #从队列的左边删除元素，并且返回删除值
- A[0] # 获取队首元素(左边第一个)
- A[-1] # 获取队尾元素(右边第一个)
```

## 3.排序sort

python中提供了两种排序函数sort和sorted，两者有明显的区别，不仅调用方式不同，而且调用之后产生的影响也不同：

`list.sort(key=None, reverse=False(or True))`

* 对原序列直接排序，对原序列产生变化，不会返回值

- 对于reverse这个bool类型参数，当reverse=False时：为正向排序；当reverse=True时：为方向排序。默认为False
- key用来定义排序规则，一般使用lambda来定义

`sorted(iterable, key=None, reverse=False(or True))`

- sorted()不会改变原来的list，而是会返回一个新的已经排序好的list

```python
>>> mylist = [(3,4),(6,3),(7,8)]
# x表示容器中的每一个元素，按照x[1]进行升序排列
>>> mylist.sort(key=lambda x:x[1],reverse=True)
>>> mylist
[(7, 8), (3, 4), (6, 3)]

>>> mylist = [2,4,3]
>>> sorted(mylist)
[2, 3, 4]
```

## 4.集合

set：集合；fronzenset：不可变集合，一旦初始化就不能添加，删除元素，可用作dict的key

**注意：Python没有提供有序集合的标准库**

`set(可迭代对象)`：接受可迭代对象，返回的是集合序列`{}`，注意与dict的区别

```python
s = set('abcc')
print(s)  # {'a', 'b', 'c'}
s = set(['a','e','d'])
print(s)  # {'a', 'd', 'e'}
fs = fronzenset('abcc')
print(fs)  # {'a', 'b', 'c'}
fs.add('b') # 出错
```

set的一些常用方法：

`add`和`pop`方法:

```
s = set()
s.add(10)
s.add(20)
s.add(30)
s.add(10)
s
Out[7]: {10, 20, 30}
s.pop()
Out[8]: 10
s
Out[9]: {20, 30}
```

`update`：将两个集合合并，相当于**并集**

```python
s = set('abcc')
another = set('bce')
s.update(another)
print(s)  # {'b', 'e', 'c', 'a'}
```

`difference`：包含在A中，但不包含在B中的元素集合，相当于**差集**

```python
s = set('abcc')
another = set('bce')
result = s.difference(another)  # 有返回值
print(result)  # {'a'}
```

集合运算：

```
result = s & another # 交集
result = s - another # 差集
result = s | another # 并集
```

判断一个元素在不在集合当中，使用in，能使用in的原因是集合实现了`__contains__`魔法方法，就可以用下面的表达式进行判断了。

```
if 'c' in s:
	return True
```

`issubset`：判断一个集合是不是另一个集合的一部分

```
>>> {'a','b'}.issubset({'a','b','c'})
True
```

### 5.字典（hashmap）

dict是一个类，它的方法都是由c语言实现。dict的优点是查询速度快，缺点是占用内存。

1.`clear()`：清空字典

2.`copy()`：返回浅拷贝（shallow copy）

3.`formkeys`：静态方法，将可迭代的对象转变成一个dict

```
new_list = ['bobby1','bobby2']
new_dict = dict.fromkeys(new_list,'age is 18')
print(new_dict)  #{'bobby1': 'age is 18', 'bobby2': 'age is 18'}

```

4.`items()`：将字典返回以元祖为元素的容器

```
d = {'a':1,'b':2}
print(type(d.items()))
for key,value in d.items():
    print(key,value)
    
输出  
<class 'dict_items'>
a 1
b 2

```

5.`setdefault(key,value)`：返回`d[‘key’]`，如果key不存在，那么设置`d[‘key’]=value`，并返回value

6.`update`：将可迭代对象组合成字典

```python
d = {'bobby':30}
d.update({'rubby':20})
print(d)  # {'bobby': 30, 'rubby': 20}
d.update(may=40)
d.update([('fubby',16)])
print(d)  # {'bobby': 30, 'rubby': 20, 'may': 40, 'fubby': 16}
d.update((('hobby',18),))
print(d) # {'bobby': 30, 'rubby': 20, 'may': 40, 'fubby': 16, 'hobby': 18}
```
