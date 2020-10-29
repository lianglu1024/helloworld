## 平衡二叉树

对于任意一个节点，左子树和右子树的高度差不能超过1

为什么要使用平衡二叉树？因为添加的数据是有序的话，搜索二叉树会退化成链表形式

以下就**不是**一颗平衡二叉树，因为节点8和12的左右子树的高度差超过了1

![image.png](https://upload-images.jianshu.io/upload_images/6853111-9c8fb28349a64808.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们将以之前实现的基于BST实现的Map为基础，添加一些自平衡的操作来完成我们的AVL树



平衡维护

* 右旋转（LL）：左子树-右子树>1 并且node.left的平衡因子>=0 

![image.png](https://upload-images.jianshu.io/upload_images/6853111-d4f9bc0ef726cf0d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

左旋转(RR)

![image.png](https://upload-images.jianshu.io/upload_images/6853111-1e9801d108b63d0c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



LR



![1529978077027](C:\Users\ZSTU\AppData\Local\Temp\1529978077027.png)

RL

![image.png](https://upload-images.jianshu.io/upload_images/6853111-86c81c6027141c2e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



基本实现

```python
class Node(object):
    def __init__(self,key=None,value=None):
        self.key = key
        self.value = value
        self.left = None
        self.right = None
        self.height = 1 # 默认节点高度是1


class AVLTree(object):
    def __init__(self):
        self.root = None
        self.size = 0

    def getSize(self):
        return self.size

    def isEmpty(self):
        return self.size == 0

    # 判断二叉树是否是一颗二分搜索树,利用中序遍历
    def isBST(self):
        keys = []
        self._inOrder(self.root,keys)
        for i in range(1,len(keys)):
            if keys[i-1] > keys[i]:
                return False
        return True

    def _inOrder(self,node,keys):
        if node is None:
            return
        self._inOrder(node.left,keys)
        keys.append(node.key)
        self._inOrder(node.right,keys)

    # 判断二叉树是否为一颗平衡二叉树
    def isBalanced(self):
        return self._isBalanced(self.root)

    def _isBalanced(self,node):
        if node is None:
            return True

        balanceFactor = self.getBalanceFactor(node)
        if abs(balanceFactor) > 1:
            return False
        return self._isBalanced(node.left) and self._isBalanced(node.right)

    # 获取节点的高度
    def getHeight(self,node):
        if node is None:
            return 0
        return node.height

    # 获得节点node的平衡因子
    def getBalanceFactor(self,node):
        if node is None:
            return 0
        return self.getHeight(node.left)-self.getHeight(node.right)

    # 对节点y进行向右旋转操作，返回旋转后新的根节点x
    #        y                              x
    #       / \                           /   \
    #      x   T4     向右旋转 (y)        z     y
    #     / \       - - - - - - - ->    / \   / \
    #    z   T3                       T1  T2 T3 T4
    #   / \
    # T1   T2
    def _rightRotate(self,y):
        x = y.left
        T3 = x.right

        # 向右旋转过程
        x.right = y
        y.left = T3

        # 更新height
        y.height = max(self.getHeight(y.left),self.getHeight(y.right)) + 1
        x.height = max(self.getHeight(x.left), self.getHeight(x.right)) + 1
        return x

    # 对节点y进行向左旋转操作，返回旋转后新的根节点x
    #    y                             x
    #  /  \                          /   \
    # T1   x      向左旋转 (y)       y     z
    #     / \   - - - - - - - ->   / \   / \
    #   T2  z                     T1 T2 T3 T4
    #      / \
    #     T3 T4
    def _leftRotate(self,y):
        x = y.right
        T2 = x.left

        # 向左旋转过程
        x.left = y
        y.right = T2

        # 更新height
        y.height = max(self.getHeight(y.left),self.getHeight(y.right)) + 1
        x.height = max(self.getHeight(x.left), self.getHeight(x.right)) + 1
        return x

    # 向二分搜索树中添加新的元素(key,value)
    def add(self,key,value):
        self.root = self._add(self.root,key,value)

    # 向以node为根的二分搜索树中插入元素(key,value)，递归算法
    # 返回插入新节点后二分搜索树的根
    def _add(self,node,key,value):
        if node is None:
            self.size+=1
            return Node(key,value)

        if key < node.key:
            node.left = self._add(node.left,key,value)
        elif key > node.key:
            node.right = self._add(node.right,key,value)
        else: # key==node.key
            node.value = value

        # 更新height
        node.height = 1 + max(self.getHeight(node.left),self.getHeight(node.right))

        # 计算平衡因子
        balanceFactor= self.getBalanceFactor(node)
        # if abs(balanceFactor) > 1:
        #     print('unbalance:',balanceFactor)

        # 平衡维护
        # LL
        if balanceFactor > 1 and self.getBalanceFactor(node.left) >= 0:
            return self._rightRotate(node)  # 右旋转
        # RR
        if balanceFactor < -1 and self.getBalanceFactor(node.right) <= 0:
            return self._leftRotate(node)  # 左旋转
        # LR
        if balanceFactor > 1 and self.getBalanceFactor(node.left) < 0:
            node.left = self._leftRotate(node.left)
            return self._rightRotate(node)
        # RL
        if balanceFactor < -1 and self.getBalanceFactor(node.right) > 0:
            node.right = self._rightRotate(node.right)
            return self._leftRotate(node)

        return node

    # 返回以node为根节点的二分搜索树中，key所在的节点
    def getNode(self,node,key):
        if node is None:
            return None

        if key == node.key:
            return node
        elif key < node.key:
            return self.getNode(node.left,key)
        else: # key > node.key
            return self.getNode(node.right,key)

    def contains(self,key):
        return self.getNode(self.root,key) != None

    def get(self,key):
        node = self.getNode(self.root,key)
        return (None if node is None else node.value)

    def set(self,key,newvalue):
        node = self.getNode(self.root,key)
        if node is None:
            raise Exception(key+"doesnot exist!")
        node.value = newvalue

    # 返回以node为根的二分搜素树的最小值所在的节点
    def mininum(self,node):
        if node.left is None:
            return node
        return self.mininum(node.left)

    # 删除掉以node为根的二分搜索树中的最小节点
    # 返回删除节点后新的二分搜索树的根
    def removeMin(self,node):
        if node.left is None:
            rightNode = node.right
            node.right = None
            self.size-=1
            return rightNode

        node.left = self.removeMin(node.left)
        return node
```

删除节点

```python
# 从二分搜索树中删除键为key的节点
def remove(self, key):
    node = self.getNode(self.root, key)
    if node is not None:
        self.root = self._remove(self.root, key)
        return node.value

    return None


def _remove(self, node, key):
    if node is None:
        return None

    if key < node.key:
        node.left = self._remove(node.left, key)
        # return node
        retNode = node

    elif key > node.key:
        node.right = self._remove(node.right, key)
        # return node
        retNode = node

    else:  # key == node.key
        # 待删除节点左子树为空的情况
        if node.left is None:
            rightNode = node.right
            node.right = None
            self.size -= 1
            # return rightNode
            retNode = rightNode

        # 待删除节点右子树为空的情况
        elif node.right is None:
            leftNode = node.left
            node.left = None
            self.size -= 1
            # return leftNode
            retNode = leftNode

        # 待删除节点左右子树均不为空的情况
        else:
            # 找到比待删除节点大的最小节点，即删除节点右子树的最小节点
            # 用这个节点顶替待删除节点的位置
            succseeor = self.mininum(node.right)
            # succseeor.right = self.removeMin(node.right)
            succseeor.right = self._remove(node.right, succseeor.key)
            succseeor.left = node.left
            node.left = node.right = None
            # return succseeor
            retNode = succseeor

    if retNode is None:
        return None

    # 更新height
    retNode.height = 1 + max(self.getHeight(retNode.left), self.getHeight(retNode.right))

    # 计算平衡因子
    balanceFactor = self.getBalanceFactor(retNode)
    # if abs(balanceFactor) > 1:
    #     print('unbalance:',balanceFactor)

    # 平衡维护
    # LL
    if balanceFactor > 1 and self.getBalanceFactor(retNode.left) >= 0:
        return self._rightRotate(retNode)  # 右旋转
    # RR
    if balanceFactor < -1 and self.getBalanceFactor(retNode.right) <= 0:
        return self._leftRotate(retNode)  # 左旋转
    # LR
    if balanceFactor > 1 and self.getBalanceFactor(retNode.left) < 0:
        retNode.left = self._leftRotate(retNode.left)
        return self._rightRotate(retNode)
    # RL
    if balanceFactor < -1 and self.getBalanceFactor(retNode.right) > 0:
        retNode.right = self._rightRotate(retNode.right)
        return self._leftRotate(retNode)

    return retNode
```



