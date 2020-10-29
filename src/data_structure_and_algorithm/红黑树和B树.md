## 红黑树

在了解红黑树之前，我们必须要知道2-3树，2-3树其实是红黑树的等价形式，同时也是了解B树的基础

### 2-3树

2-3树中，每个节点包含1个或者2个元素，有一个元素的称为2节点（因为它可以有两个孩子），有两个元素的称为3节点（因为它可以有三个孩子）。2-3树是绝对平衡的树结构。

![](https://ws2.sinaimg.cn/large/006tNbRwgy1fup77gpof7j30gd08s760.jpg)

* 插入元素时，选择的都是插入到叶子结点上

* 如果是插入到2结点上，我们直接融合；如果插入到的是三节点，我们先融合，在去拆分，记住拆分的元素位于中间。

![](https://ws4.sinaimg.cn/large/006tNbRwgy1fup7dv5jsnj31kw0vyqii.jpg)

![](https://ws3.sinaimg.cn/large/006tNbRwgy1fup7egr7x6j31kw0t1ngf.jpg)

### 红黑树

![](https://ws1.sinaimg.cn/large/006tNbRwgy1fup7fybafpj31fc0ucn9e.jpg)

![](https://ws4.sinaimg.cn/large/006tNbRwgy1fup7glosomj31kw0m0wsk.jpg)

因此我们得出关于红黑树的特性：

* 每个结点要么是红色的，要么是黑色的
* 根节点是黑色的
* 每个叶子结点（最后的空节点）是黑色的
* 如果一个结点是红色的，那么他的孩子结点都是黑色的
* 从任意一个结点到叶子结点，经过的黑色结点是一样的

**结论：红黑树是保持“黑平衡”的二叉树，不是平衡二叉树；红黑树适合添加，删除元素，因为这些操作相比较于平衡二叉树简单；平衡二叉树适用于查找**

## B树

B树的插入操作：http://www.iqiyi.com/w_19rv78y589.html



**B树和B+树的区别：B+树将数据存在叶子结点里，非叶子结点存索引，并且叶子结点之间通过链表相连**

B树

![](https://ws4.sinaimg.cn/large/006tNbRwgy1fup7ttvqubj30m807qmy5.jpg)

B+树

![](https://ws2.sinaimg.cn/large/006tNbRwgy1fup7u3dp0rj30m8091wfh.jpg)

## 为什么使用 B+树

1. B+树更适合外部存储,由于内节点无 data 域,一个结点可以存储更多的内结点,每个节点能索引的范围更大更精确,也意味着 B+树单次磁盘IO的信息量大于B-树,I/O效率更高。
2. Mysql是一种关系型数据库，区间访问是常见的一种情况，B+树叶节点增加的链指针,加强了区间访问性，可使用在范围区间查询等，而B-树每个节点 key 和 data 在一起，则无法区间查找

B树相关的博文：

https://segmentfault.com/a/1190000004690721

https://www.jianshu.com/p/4dbbaaa200c4

http://www.niceru.com/topic/420.html

