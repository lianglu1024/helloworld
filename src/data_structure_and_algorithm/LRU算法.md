# LRU
LRU（Least recently used，最近最少使用）算法根据数据的历史访问记录来进行淘汰数据，其核心思想是“如果数据最近被访问过，那么将来被访问的几率也更高”。

> Design and implement a data structure for Least Recently Used (LRU) cache. It should support the following operations: get and set.
> 
> get(key) - Get the value (will always be positive) of the key if the key exists in the cache, otherwise return -1.
> set(key, value) - Set or insert the value if the key is not already present. When the cache reached its capacity, it should > invalidate the least recently used item before inserting a new item.


```python
class LRUCache:

    def __init__(self, capacity):
        self.cache = {}
        self.used_list = []
        self.capacity = capacity

    def get(self, key):
        if key in self.cache:
            if key != self.used_list[-1]:
                self.used_list.remove(key)
                self.used_list.append(key)
            return self.cache[key]
        else:
            return -1

    def set(self, key, value):
        if key in self.cache:
            self.used_list.remove(key)
        elif len(self.cache) == self.capacity:
            self.cache.pop(self.used_list.pop(0))
        self.used_list.append(key)
        self.cache[key] = value
```
