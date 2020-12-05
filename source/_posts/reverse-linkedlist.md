---
title: 单链表反转
mathjax: true
date: 2020-12-05 16:01:11
excerpt: 单链表反转的几种方法，知识点来自剑指offer06、24
categories: 算法笔记
tags: ['链表','数据结构']
keywords: ['linkedlist','链表反转']
---



> 前几天做题遇到了链表反转的问题，即[剑指offer06](https://leetcode-cn.com/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/)和[24](https://leetcode-cn.com/problems/fan-zhuan-lian-biao-lcof/)，涉及到了链表反转的知识。奈何本人太菜，早已忘了数据结构课本中链表反转的具体操作，这里记录并复习一下有关单向链表反转的几种方法。
>
> 参考链接：[单链表反转详解](http://c.biancheng.net/view/8105.html)



# 定义

链表节点定义如下(python)：

```python
class ListNode:		# 定义链表节点
    def __init__(self, x):
        self.val = x
        self.next = None
```

原链表：1->2->3->4->5->NULL

反转后：5->4->3->2->1->NULL



# 新建链表法

新建链表法的主要思想是新建一个链表，保证其与原链表顺序相反，然后返回新链表，实现原链表的反转。新建反转链表有两种方法，一种是使用头插法新建链表，另一种是使用辅助栈(或者数组等)。这类方法的主要特点是需要使用额外的存储空间。



## 头插法

使用头插法新建一个链表(虽说是新链表，只是换了个头指针，但没有使用新的存储空间)，每次将新节点插入到新链表头节点之后，然后返回新链表实现反转。

```python
def reverseList(self, head: ListNode) -> ListNode:
    if not head or not head.next: return head
    new_head = None         # 新链表的头指针
    tmp = None				# 临时指针tmp，用于保存原链表的当前头节点
    while head:
        tmp = head			# 指向当前头节点
        head = head.next	# 当前头指针后移。这条语句一定要在tmp.next=new_head的前面

        # 将tmp节点加入到新链表的头节点后面(头插法)
        tmp.next = new_head 
        new_head = tmp
    return new_head
```

- 时间复杂度$O(N)$：遍历链表使用线性大小空间。
- 空间复杂度$O(1)$：只需要两个额外的指针，没有额外使用存储空间



## 辅助栈新建链表

第二种方式是使用辅助栈的方法，将原链表依次入栈，然后新建链表，将栈中的元素依次出栈作为新链表的下一个节点，则新链表即为原链表的反转链表。

```python
def reverseList(self, head: ListNode) -> ListNode:
    if not head or not head.next: return head
    stack = []   	# 用列表表示栈
    while head:		# 将原链表的值依次入栈
        stack.append(head.val)
        head = head.next
    new_head = ListNode(stack.pop())  # 注意这里的头节点是有值的
    cur = new_head		# cur用于尾插法构建新链表，理解为向后移动的指针
    for i in range(len(stack)): 	# 将栈中数据出栈，并构建新链表
        cur.next = ListNode(stack.pop())
        cur = cur.next
    return new_head			# 返回新链表。将new_head和cur都理解为指针
```

- 时间复杂度$O(N)$：遍历链表使用线性大小空间。
- 空间复杂度$O(N)$：额外辅助空间和新建链表的存储空间，均为线性大小。



# 迭代遍历法(双指针)

迭代反转法(原地反转法)，从当前链表的头结点开始，一直遍历到链表的最后一个节点，期间逐个改变节点指针的方向，使其指向前一个节点，返回最后一个位置的指针(或者说将头指针指向最后一个位置)。参考[链接](https://leetcode-cn.com/problems/fan-zhuan-lian-biao-lcof/solution/jian-zhi-offer-24-fan-zhuan-lian-biao-die-dai-di-2/)

```python
def reverseList(self, head: ListNode) -> ListNode:
    if not head or not head.next: return head
    pre,cur = None, head  # pre初始化为空节点，可以理解为头节点的前一个节点
    while cur:
        tmp = cur.next	# 暂时存放当前节点的后继节点
        cur.next = pre	# 修改当前节点的指针方向
        pre = cur		# pre暂存cur(pre右移)
        cur = tmp		# cur指针右移，访问下一节点
    return pre  		# 最后返回pre指针，作为头指针，或者说是将头指针指向pre，实现反转。 
```

- 时间复杂度$O(N)$：遍历链表使用线性大小空间。
- 空间复杂度$O(1)$：变量`pre`和`cur`使用常数大小额外空间。



# 就地逆置法

就地逆置法(也叫头插法)，与头插法新建链表类似，不同的是不需要新链表，只需要用两个指针对原链表本身进行操作，将每个节点依次插入到头节点后面，最后返回头节点。

```python
def reverseList(self, head: ListNode) -> ListNode:
    if not head or not head.next: return head
    pre, cur = head, head.next
    while cur:
        pre.next = cur.next		# 将cur指向的节点"摘除"
        cur.nxet = head		# 将上一步“摘除”的节点放到head的前面，此时头节点为cur指向的节点
        head = cur		# head更新，重新指向头节点
        cur = pre.next		# cur重新指向pre的next节点
    return head
```

> 此方法和上面提到的头插法大同小异，一个是返回原来的头指针,一个是返回新指针。此方法中的**原链表头指针不动**，上面的头插法的原链表头指针是移动的。
>
> 这两种方法都可以叫做头插法。

- 时间复杂度$O(N)$：遍历链表使用线性大小空间。
- 空间复杂度$O(1)$：变量`pre`和`cur`使用常数大小额外空间。



# 递归法

递归法的思想是递归到停止条件，即链尾，然后逐层返回递归结果(从后向前将链表的指向反向)。

```python
def reverseList(self, head: ListNode) -> ListNode:
    if not head or not head.next: return head
    new_head = self.reverseList(head.next)   # 一直递归，找到链表中最后一个节点
    
    # 当逐层退出时，new_head 的指向都不变，一直指向原链表中最后一个节点；
    # 递归每退出一层，函数中 head 指针的指向都会发生改变，都指向上一个节点。
    
    # 每退出一层，都需要改变 head->next 节点指针域的指向，同时令 head 所指节点的指针域为None。
    head.next.next = head
    head.next = None
    
    # 每一层递归结束，都要将新的头指针返回给上一层。
    # 由此，即可保证整个递归过程中，能够一直找得到新链表的表头。
    return new_head
    
```

- 时间复杂度$O(N)$：一遍递归深入并返回
- 空间复杂度$O(1)$：不需要额外空间，只需要两个指针(但迭代过程需要一直创建/释放)

> 这递归太难理解了 :-( ，建议看大佬题解[剑指offer24题解](https://leetcode-cn.com/problems/fan-zhuan-lian-biao-lcof/solution/dong-hua-yan-shi-duo-chong-jie-fa-206-fan-zhuan-li/)或者[单链表反转详解](http://c.biancheng.net/view/8105.html)。



