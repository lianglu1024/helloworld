## 二叉树

二叉树的基本形状和基本概念

![image.png](https://upload-images.jianshu.io/upload_images/6853111-51c0747a68478cbd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 节点的**度**：节点拥有的子树数，例如28的度为2,13的度为0
* 节点的**层次**：以根（Root）节点为第一层，根的孩子为第二次，以此类推。如节点13处在第三层
* 树的**深度**或**高度**：树中节点的最大层次。
* **满二叉树**：一棵深度为k，且有2^k-1个节点的树是满二叉树。 例如上图

* **完全二叉树**：一棵二叉树，除了最后一层之外都是完全填充的，并且最后一层的叶子结点都在左边。满二叉树一定是完全二叉树，完全二叉树不一定是满二叉树。 如下图：

  ![image.png](https://upload-images.jianshu.io/upload_images/6853111-70515f79eccc765d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 二分搜索树（BST）

二分搜索树是一种**有序**的树结构。

![image.png](https://upload-images.jianshu.io/upload_images/6853111-1c5ccc1e56951677.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

实现二分搜索树的主要方法：

1.`getSize`：获取树中节点的个数

2.`add`：向树中添加元素

3.`contains`：查看二分搜素树是否包含元素e

4.`preOrder`，`inOrder`，`postOrder`：前序，中序，后序遍历二分搜索树

5.`preOrderNR`：二分搜索树的前序遍历的非递归实现

6.`levelOrder`：层序遍历（广度优先）

7.`minimum`，`maximum`：获取最小（大）节点

8.`removeMin`，`removeMax`：删除最小（大）节点

9.`remove`：删除指定节点



### 一.基本方法实现

```python
class Node(object):
    def __init__(self,e=None):
        self.e = e
        self.left = None
        self.right = None


class BST(object):
    def __init__(self):
        self.root = None
        self.size = 0

    def getSize(self):
        return self.size

    def isEmpty(self):
        return self.size == 0

    # 向二分搜素树中添加新的元素e
    def add(self,e):
        self.root = self._add(self.root,e)

    # 向以node为根的二分搜索树中插入元素e，递归算法
    # 返回插入新节点后二分搜索树的根
    def _add(self,node,e):
        if node is None:
            self.size+=1
            return Node(e)
        if e < node.e:
            node.left = self._add(node.left,e)
        elif e > node.e:
            node.right = self._add(node.right,e)
        return node

    # 查看二分搜素树是否包含元素e
    def contains(self,e):
        return self._contains(self.root,e)

    # 查看以node为根的二分搜索树是否包含元素e，递归算法
    def _contains(self,node,e):
        if node is None:
            return False
        if e == node.e:
            return True
        elif e< node.e:
            return self._contains(node.left,e)
        else:
            return self._contains(node.right,e)
```

### 二.前序，中序，后序遍历

![image.png](https://upload-images.jianshu.io/upload_images/6853111-5b079d569d1168d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





**<u>中序遍历得到的是有序的数列</u>**。因此我们可以通过将二叉树通过中序遍历之后来验证该树是不是二叉搜索树

```python
# 前序遍历 根->左->右
def preOrder(self):
    self._preOrder(self.root)

def _preOrder(self,node):
    if node is None:
        return
    print(node.e)
    self._preOrder(node.left)
    self._preOrder(node.right)


# 中序遍历 左->中->右
def inOrder(self):
    self._inOrder(self.root)

def _inOrder(self,node):
    if node is None:
        return
    self._inOrder(node.left)
    print(node.e)
    self._inOrder(node.right)

# 后序遍历 左->右->中
def postOrder(self):
    self._postOrder(self.root)

def _postOrder(self,node):
    if node is None:
        return
    self._postOrder(node.left)
    self._postOrder(node.right)
    print(node.e)
```

### 三.前序遍历的非递归实现

需要借助**栈**来实现非递归的遍历

1. 先将28（根）`push`到栈中，然后`pop`出栈，查看28是否有右节点和左节点
2. 将左节点30和右节点16`push`到栈中（因为栈是后进先出，所以先出的是右节点），然后`pop`出栈。查看16是否有左节点和右节点
3. 将左节点22和右节点13`push`到栈中，然后`pop`出栈。查看13是否有左节点和右节点
4. 因为13没有左右节点，接着`pop`，查看22是否有左右节点，这样以此类推下去.......
5. 最终`pop`出的元素就是前序遍历的结果

![image.png](https://upload-images.jianshu.io/upload_images/6853111-9f6c4d3d68cf51aa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```python
# 二叉搜索树的非递归前序遍历
def preOrferNR(self):
    stack = []  # 使用list来做栈
    stack.append(self.root)
    while len(stack) != 0:
        cur = stack.pop()
        print(cur.e)

        if cur.right != None:
            stack.append(cur.right)
        if cur.left != None:
            stack.append(cur.left)
```

### 三.二分搜索树的层序遍历

需要借助**队列**来完成

1. 先将28（根）入队，然后`dequeue`，查看28是否有右节点和左节点。结果有左右节点。
2. 将右节点（16）和左节点（30）都入队，然后`dequeue`（出队的是16），查看16是否有右节点和左节点。结果有左右节点
3. 将右节点（13）和左节点（22）都入队，然后`dequeue`（出队的是30），查看30是否有右节点和左节点。结果有左右节点
4. 这样以此类推下去.......
5. 最终`dequeue`的元素就是层序遍历的结果



![image.png](https://upload-images.jianshu.io/upload_images/6853111-ec1b71706d7b901e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```python
# 二分搜索树的层序遍历
def levelOrder(self):
    from queue import Queue
    q = Queue()
    q.put(self.root)
    while not q.empty():
        cur = q.get()
        print(cur.e)

        if cur.left is not None:
            q.put(cur.left)
        if cur.right is not None:
            q.put(cur.right)
```

补充：关于python自带的queue的常见用法

```python
from Queue import queue
q = queue()
q.put(2)   # 入队
q.get()  # 出队
q.gsize()  # 获取队列中元素个数
q.empty()  # 队列是否为空，返回布尔值
```

### 四.二分搜索树的最小值和最大值

最小值处在叶子节点的最右侧；最大值处在叶子节点的最左侧。

注意：当我们查找最小（大）值的时候，就跟左（右）子树没有任何的关系了

![image.png](https://upload-images.jianshu.io/upload_images/6853111-ec234ef532eb467c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```python
# 寻找二分搜索树的最小元素
def mininum(self):
    if self.size == 0:
        raise Exception("BST is empty")

    return self._mininum(self.root).e

# 返回以node为根的二分搜索树的最小值所在的节点
def _mininum(self, node):
    if node.left is None:
        return node
    return self._mininum(node.left)


# 寻找二分搜索树的最大元素
def maxinum(self):
    if self.size == 0:
        raise Exception("BST is empty")
    return self._maxinum(self.root).e

# 返回以node为根的二分搜索树的最大值所在的节点
def _maxinum(self, node):
    if node.right is None:
        return node
    return self._mininum(node.right)
```



### 五.删除二分搜索树的最大（小）值

如果是叶子节点，直接删除即可；如果是非叶子节点，需要将整个子树移植到最大（小）元素位置。针对第二种情况，例如要删除最大元素58，则需把58的右子树移植过来（注意，最大元素没有左子树；最小元素没有右子树）

![image.png](https://upload-images.jianshu.io/upload_images/6853111-e813ea3eed462199.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/6853111-e16a0b32bba43d48.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

方法很像add的方法

```python
# 从二分搜索树中删除最小值所在的节点，返回最小值
def removeMin(self):
    ret = self.mininum()
    self.root = self._removeMin(self.root)
    return ret

# 删除以node为根的二分搜索树中的最小节点
# 返回删除节点后新的二分搜索树的根
def _removeMin(self, node):
    if node.left is None:
        rightNode = node.right
        node.right = None
        self.size -= 1
        return rightNode

    node.left = self._removeMin(node.left)
    return node


# 从二分搜索树中删除最大值所在的节点，返回最大值
def removeMax(self):
    ret = self.maxinum()
    self.root = self._removeMax(self.root)
    return ret


# 删除以node为根的二分搜索树中的最大节点
# 返回删除节点后新的二分搜索树的根
def _removeMax(self, node):
    if node.right is None:
        leftNode = node.left
        node.left = None
        self.size -= 1
        return leftNode

    node.right = self._removeMax(node.right)
    return node
```



### 六.删除任意节点

删除只有左（右）孩子的节点，只需移植子树到节点上去

删除即有左孩子，又有右孩子的节点，如下

![image.png](https://upload-images.jianshu.io/upload_images/6853111-5e807f3843260c1f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1962年，Hibbard提出Hibbard Deletion：更新节点的右（左）子树，与左（右）子树没有关系。当然这不是绝对的，只是这样子比较方便操作。如何查找出后继元素呢？后继元素s即为d的右子树中最小节点。

![image.png](https://upload-images.jianshu.io/upload_images/6853111-f7859da8c2afdde6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/6853111-8d0c6c9bb8e923f8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```python
# 从二分搜索树中删除元素为e的节点
def remove(self, e):
    self.root = self._remove(self.root, e)


# 删除以node为根的二分搜索树中值为e的节点，递归算法
# 返回删除节点后新的二分搜索树的根
def _remove(self, node, e):
    if node is None:
        return None

    if e < node.e:
        node.left = self._remove(node.left, e)
        return node
    elif e > node.e:
        node.right = self._remove(node.right, e)
        return node
    else:  # e==node.e
        # 待删除节点左子树为空的情况
        if node.left is None:
            rightNode = node.right
            node.right = None
            self.size -= 1
            return rightNode

    # 待删除节点左右子树均不为空的情况
    # 找到比待删除节点大的最小节点，即待删除节点右子树的最小节点
    # 用这个节点顶替待删除节点的位置
    successor = self.mininum(node.right)
    successor.right = self.removeMin(node.right)
    successor.left = node.left

    node.left = node.right = None
    return successor
```

## 已知两种遍历序列求原始二叉树

### 需要明确的前提条件

- 通过先序和中序可以求出原始二叉树
- 通过中序和后序可以求出原始二叉树
- 但是通过先序和后序**无法**还原出二叉树

**换种说法：**

- 只有通过**先序中序或者后序中序**才可以确定一个二叉树

一、已知先序和中序求后序

已知先序为：ABCDEFGH 中序为：BDCEAFHG

1.先序的第一位为根节点，所以A为根节点。

2.中序中根节点左边为左子树，右边为右子树，所以BDCE为左子序列，FHG为右子序列

3.先序中左子序列先出现的为子序列的根节点，所以B为左子序列的根节点

4.再次回到中序序列中，左子序列的根节点左边为其左子序列，右边为其右子序列，所以子序列的根节点无左子序列，右子序列为DCE。

5.同理，推测下一阶的子序列和根节点的右序列

![由遍历序列恢复二叉树](http://s15.sinaimg.cn/mw690/0065K84dzy73W56LF9A3e&690)

​      由此推出后序为：DECBHGFA

二、已知中序和后序求先序：

已知中序为：BDGEAFHG 中后序为：DECBHGFA

1.后序的最后一个为根节点，所以A为根节点

2.中序中根节点左边为左子树，右边为右子树，所以BDGE为左子树，FGH为右子树

3.后序中左子序列的中最后出现的为左子序列的根节点，所以B为左子序列的根节点

4.再次回到中序中，左子序列根节点的左侧为其左子序列，右侧为其右子序列，所以B子序列无左子序列，右子序列为DEG。

5.同理，推测出B的其他子序列以及跟节点的整个右子序列。

![由遍历序列恢复二叉树](http://s3.sinaimg.cn/mw690/0065K84dzy73W5N1xZgc2&690)

 由此推出先序为：ABCDEFGH

