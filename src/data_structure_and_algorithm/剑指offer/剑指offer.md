## 1.斐波那契数列
大家都知道斐波那契数列，现在要求输入一个整数n，请你输出斐波那契数列的第n项（从0开始，第0项为0，第1项是1）。n<=39

递归解法：
```python
class Solution:
    def Fibonacci(self, n):
        # write code here
        if n == 0:
            return 0
        if n == 1:
            return 1
        if n == 2:
            return 1
        return self.Fibonacci(n-1) + self.Fibonacci(n-2)
```
记忆法：
```python
class Solution:
    def Fibonacci(self, n):
        # write code here
        if n == 0: 
            return 0
        if n == 1:
            return 1
        if n == 2:
            return 1
        a = 1
        b = 1
        for i in range(3, n+1):
            c = a + b
            a = b
            b = c
        return c
```
## 2.跳台阶
一只青蛙一次可以跳上1级台阶，也可以跳上2级。求该青蛙跳上一个n级的台阶总共有多少种跳法（先后次序不同算不同的结果）。

解题思路：要想跳到第n级台阶，那么就要从第n-1级台阶或者n-2级台阶开始跳

记忆法：
```python
class Solution:
    def jumpFloor(self, number):
        # write code here
        if number == 0:
            return 0
        if number == 1:
            return 1
        if number == 2:
            return 2
        a = 1
        b = 2
        for i in range(3, number+1):
            c = a + b
            a = b
            b = c
        return c
```
## 3.变态跳台阶
一只青蛙一次可以跳上1级台阶，也可以跳上2级……它也可以跳上n级。求该青蛙跳上一个n级的台阶总共有多少种跳法。

解题思路：把公式写出来，可以化简

```python
class Solution:
    def jumpFloorII(self, number):
        # write code here
        if number == 0:
            return 0
        if number == 1:
            return 1
        if number == 2:
            return 2
        a = 2
        for i in range(3, number+1):
            a = 2 * a
        return a
```
## 4.矩形覆盖
我们可以用`2*1`的小矩形横着或者竖着去覆盖更大的矩形。请问用n个`2*1`的小矩形无重叠地覆盖一个`2*n`的大矩形，总共有多少种方法？比如n=3时，`2*3`的矩形块有3种覆盖方法

```python
class Solution:
    def rectCover(self, number):
        # write code here
        if number == 0:
            return 0
        if number == 1:
            return 1
        if number == 2:
            return 2
        a = 1
        b = 2
        for i in range(3, number+1):
            c = a + b
            a = b
            b = c
        return c
```
## 5.二维数组中的查找
在一个二维数组中（每个一维数组的长度相同），每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

解题思路：每一行的最后一个元素是该行的最大值，第一行的元素是该列的最小值，所以遍历顺序是从第一行的最后一个元素开始向左，然后向下遍历
```python
class Solution:
    # array 二维列表
    def Find(self, target, array):
        # write code here
        m = len(array)
        n = len(array[0])
        i = m - 1
        j = 0
        while 0 <= i <= m-1 and 0 <= j <= n-1:
            if array[i][j] == target:
                return True
            elif array[i][j] > target:
                i-=1
            else:
                j+=1
        return False
```

## 6.替换空格
请实现一个函数，将一个字符串中的每个空格替换成“%20”。例如，当字符串为We Are Happy.则经过替换之后的字符串为We%20Are%20Happy。
```python
class Solution:
    # s 源字符串
    def replaceSpace(self, s):
        # write code here
        result = []
        for c in s:
            if c == " ":
                result.append("%")
                result.append("2")
                result.append("0")
            else:
                result.append(c)
        return "".join(result)
```

## 7.用两个栈实现队列
用两个栈来实现一个队列，完成队列的Push和Pop操作。 队列中的元素为int类型。
```python
class Solution:
    def __init__(self):
        self.stack1 = []
        self.stack2 = []
    def push(self, node):
        # write code here
        self.stack1.append(node)
    def pop(self):
        # return xx
        if self.stack2:
            return self.stack2.pop()
        else:
            for i in range(len(self.stack1)):
                self.stack2.append(self.stack1.pop())
        return self.stack2.pop()
```

## 8.旋转数组的最小数字
把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。输入一个非递减排序的数组的一个旋转，输出旋转数组的最小元素。

例如数组{3,4,5,1,2}为{1,2,3,4,5}的一个旋转，该数组的最小值为1。

NOTE：给出的所有元素都大于0，若数组大小为0，请返回0。
```python
class Solution:
    def minNumberInRotateArray(self, rotateArray):
        # write code here
        if not rotateArray:
            return 0
        l = len(rotateArray)
        if l == 1:
            return rotateArray[0]
        for i in range(1, l):
            if rotateArray[i] < rotateArray[i-1]:
                return rotateArray[i]
```
 方法二：二分查找
 ```python
 ?
 ```
 ## 9.数组中重复的数字
 在一个长度为n的数组里的所有数字都在0到n-1的范围内。 数组中某些数字是重复的，但不知道有几个数字是重复的。也不知道每个数字重复几次。请找出数组中任意一个重复的数字。 例如，如果输入长度为7的数组{2,3,1,0,2,5,3}，那么对应的输出是第一个重复的数字2。

 哈希表：
 ```python
 class Solution:
    # 这里要特别注意~找到任意重复的一个值并赋值到duplication[0]
    # 函数返回True/False
    def duplicate(self, numbers, duplication):
        # write code here
        d = {}
        for n in numbers:
            if d.get(n) is None:
                d[n] = 1
            else:
                duplication[0] = n
                return True
        return False
 ```
 方法二（空间复杂度O(1)）：利用数字都在0到n-1的范围内
 ```python
 
 ```
 ## 10.调整数组顺序使奇数位于偶数前面
输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有的奇数位于数组的前半部分，所有的偶数位于数组的后半部分，并保证奇数和奇数，偶数和偶数之间的相对位置不变。

## 11.包含min函数的栈
定义栈的数据结构，请在该类型中实现一个能够得到栈中所含最小元素的min函数（时间复杂度应为O（1））。

解题思路：
| 步骤 | 操作  | 数据栈     | 辅助栈     | 最小值 |
| ---- | ----- | ---------- | ---------- | ------ |
| 1    | 压入3 | 3          | 3          | 3      |
| 2    | 压入4 | 3，4       | 3，3       | 3      |
| 3    | 压入2 | 3，4，2    | 3，3，2    | 2      |
| 4    | 压入1 | 3，4，2，1 | 3，3，2，1 | 1      |
| 5    | pop   | 3，4，2    | 3，3，2    | 2      |
| 6    | pop   | 3，4       | 3，3       | 3      |
| 7    | 压入0 | 3，4，0    | 3，3，0    | 0      |


```python
class Solution:
    def __init__(self):
        self.stack = []
        self.helper = []
    def push(self, node):
        # write code here
        self.stack.append(node)
        if not self.helper:
            self.helper.append(node)
            return
        if node <= self.helper[-1]:
            self.helper.append(node)
        else:
            self.helper.append(self.min())
    def pop(self):
        # write code here
        self.helper.pop()
        return self.stack.pop()
    def top(self):
        # write code here
        return self.stack[-1]
    def min(self):
        # write code here
        return self.helper[-1]
```

## 12.栈的压入、弹出序列
输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否可能为该栈的弹出顺序。假设压入栈的所有数字均不相等。例如序列1,2,3,4,5是某栈的压入顺序，序列4,5,3,2,1是该压栈序列对应的一个弹出序列，但4,3,5,1,2就不可能是该压栈序列的弹出序列。（注意：这两个序列的长度是相等的）

注意：保证测试中不会当栈为空的时候，对栈调用pop()或者min()或者top()方法。

解题思路：借助临时栈，模拟栈的压入和弹出
```python
class Solution:
    def IsPopOrder(self, pushV, popV):
        # write code here
        tmp = []
        l = len(pushV)
        i = 0
        j = 0
        while i < l and j < l:
            if popV[j] != pushV[i]:
                tmp.append(pushV[i])
                i+=1
                continue
            else:
                i+=1
                j+=1
        for c in tmp:
            if c != popV.pop():
                return False
        return True
```

## 13.最小的K个数
输入n个整数，找出其中最小的K个数。例如输入4,5,1,6,2,7,3,8这8个数字，则最小的4个数字是1,2,3,4,。

解题思路：最小堆
```python
class Solution:
    def GetLeastNumbers_Solution(self, tinput, k):
        # write code here
        import heapq
        heap = []
        if len(tinput) < k:
            return []
        for i in tinput:
            heapq.heappush(heap, i)
        result = []
        for i in range(k):
            result.append(heapq.heappop(heap))
        return result
```
## 14.从尾到头打印链表

解题思路：利用栈的先进后出的特点
输入一个链表，按链表从尾到头的顺序返回一个ArrayList。
```python
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution:
    # 返回从尾部到头部的列表值序列，例如[1,2,3]
    def printListFromTailToHead(self, listNode):
        # write code here
        p = listNode
        tmp_stack = []
        res = []
        while p:
            tmp_stack.append(p.val)
            p = p.next
        for i in range(len(tmp_stack)):
            res.append(tmp_stack.pop())
        return res
```

## 15.链表中倒数第k个结点
输入一个链表，输出该链表中倒数第k个结点。

解题思路：前后指针
```python
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution:
    def FindKthToTail(self, head, k):
        # write code here
        if not head:  # 边界
            return None
        p1 = head
        p2 = head
        for i in range(k):
            if not p2:  # 边界
                return None
            p2 = p2.next
        while p2:
            p2 = p2.next
            p1 = p1.next
        return p1
```

## 16.反转链表
输入一个链表，反转链表后，输出新链表的表头。
```python
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None
class Solution:
    # 返回ListNode
    def ReverseList(self, pHead):
        # write code here
        if not pHead:
            return None
        res = []
        while pHead:
            res.append(pHead)
            pHead = pHead.next
        for i in range(len(res)-1):
            if i == 0:
                res[i].next = None
            res[i+1].next = res[i]
        return res[len(res)-1]
```
## 17.合并两个排序的链表
输入两个单调递增的链表，输出两个链表合成后的链表，当然我们需要合成后的链表满足单调不减规则。
```python
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None
class Solution:
    # 返回合并后列表
    def Merge(self, pHead1, pHead2):
        # write code here
        res = []
        while pHead1 and pHead2:
            if pHead1.val > pHead2.val:
                res.append(pHead2)
                pHead2 = pHead2.next
            else:
                res.append(pHead1)
                pHead1 = pHead1.next
        if pHead1 is None:
            while pHead2:
                res.append(pHead2)
                pHead2 = pHead2.next
        else:
            while pHead1:
                res.append(pHead1)
                pHead1 = pHead1.next
        for i in range(len(res)):
            if i == len(res)-1:
                res[i].next = None
                break
            res[i].next = res[i+1]
        if not res:
            return None
        return res[0]
```

## 18.复杂链表的复制
输入一个复杂链表（每个节点中有节点值，以及两个指针，一个指向下一个节点，另一个特殊指针random指向一个随机节点），请对此链表进行深拷贝，并返回拷贝后的头结点。（注意，输出结果中请不要返回参数中的节点引用，否则判题程序会直接返回空）

## 19.两个链表的第一个公共结点
输入两个链表，找出它们的第一个公共结点。（注意因为传入数据是链表，所以错误测试数据的提示是用其他方式显示的，保证传入数据是正确的）

## 20.孩子们的游戏(圆圈中最后剩下的数)
每年六一儿童节,牛客都会准备一些小礼物去看望孤儿院的小朋友,今年亦是如此。HF作为牛客的资深元老,自然也准备了一些小游戏。其中,有个游戏是这样的:首先,让小朋友们围成一个大圈。然后,他随机指定一个数m,让编号为0的小朋友开始报数。每次喊到m-1的那个小朋友要出列唱首歌,然后可以在礼品箱中任意的挑选礼物,并且不再回到圈中,从他的下一个小朋友开始,继续0...m-1报数....这样下去....直到剩下最后一个小朋友,可以不用表演,并且拿到牛客名贵的“名侦探柯南”典藏版(名额有限哦!!^_^)。请你试着想下,哪个小朋友会得到这份礼品呢？(注：小朋友的编号是从0到n-1)

如果没有小朋友，请返回-1

## 21.链表中环的入口结点
给一个链表，若其中包含环，请找出该链表的环的入口结点，否则，输出null。

## 22.删除链表中重复的结点
在一个排序的链表中，存在重复的结点，请删除该链表中重复的结点，重复的结点不保留，返回链表头指针。 例如，链表1->2->3->3->4->4->5 处理后为 1->2->5
