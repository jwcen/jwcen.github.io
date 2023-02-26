---
layout: post
title:  "[数据结构] 01-链表相关"
categories: [数据结构与算法]
tags: [链表]
permalink: /posts/:title:output_ext
description: 链表、循环链表、双向链表、环形链表
math: true
---

## 设计链表
{% details 初始化链表代码 %}
~~~go
func InitList(n int) *LinkedNode {
    dummy := &LinkedNode{}
    pre := dummy
    for i := 1; i <= n; i++ {
        cur := &LinkedNode{Val: i}
        pre.Next = cur
        pre = pre.Next
    }
    return dummy.Next
}
~~~
{% enddetails %}

### [707. 设计链表](https://leetcode.cn/problems/design-linked-list/description/)
{% details 题目详情 %}

设计链表的实现。您可以选择使用单链表或双链表。单链表中的节点应该具有两个属性：val 和 next。val 是当前节点的值，next 是指向下一个节点的指针/引用。
如果要使用双向链表，则还需要一个属性 prev 以指示链表中的上一个节点。假设链表中的所有节点都是 0-index 的。
在链表类中实现这些功能：
- get(index)：获取链表中第 index 个节点的值。如果索引无效，则返回-1。
- addAtHead(val)：在链表的第一个元素之前添加一个值为 val 的节点。插入后，新节点将成为链表的第一个节点。
- addAtTail(val)：将值为 val 的节点追加到链表的最后一个元素。
- addAtIndex(index,val)：在链表中的第 index 个节点之前添加值为 val  的节点。如果 index 等于链表的长度，则该节点将附加到链表的末尾。如果 index 大于链表长度，则不会插入节点。如果index小于0，则在头部插入节点。
- deleteAtIndex(index)：如果索引 index 有效，则删除链表中的第 index 个节点。
{% enddetails %}

{% details 单链表实现 go %}
~~~go
type Node struct {
	Val  int
	Next *Node
}

// NewNode 创建一个节点
func NewNode(val int) *Node {
	return &Node{Val: val, Next: nil}
}

type LinkedList struct {
	Header *Node // 虚拟头节点
	Len   int
}

// Constructor 创建一个链表
func Constructor() LinkedList {
	header := NewNode(0)
	return LinkedList{Header: header, Len: 0}
}

// Get：获取链表中第 index 个节点的值。如果索引无效，则返回-1
func (l *LinkedList) Get(index int) int {
	if index < 0 || index > l.Len-1 {
		return -1
	}

	cur := l.Header
	for i := 0; i <= index; i++ {
		cur = cur.Next
	}
	return cur.Val
}

// AddAtHead(val)：在链表的第一个元素之前添加一个值为 val 的节点。
// 插入后，新节点将成为链表的第一个节点。
func (l *LinkedList) AddAtHead(val int) {
	// 在Header节点后面直接添加一个节点
	node := NewNode(val)
	node.Next = l.Header.Next
	l.Header.Next = node

	l.Len++
}

// AddAtTail：将值为 val 的节点追加到链表的最后一个元素。
func (l *LinkedList) AddAtTail(val int) {
	node := NewNode(val)
	cur := l.Header
	for cur.Next != nil {
		cur = cur.Next
	}

	cur.Next = node
	l.Len++
}

// AddAtIndex：在链表中的第 index 个节点之前添加值为 val  的节点。
// 如果 index 等于链表的长度，则该节点将附加到链表的末尾。
// 如果 index 大于链表长度，则不会插入节点。如果index小于0，则在头部插入节点。
func (l *LinkedList) AddAtIndex(index int, val int) {
    if index > l.Len {
		return
	}
	if index == l.Len {
		l.AddAtTail(val)
		return
	} 
    if index < 0 {
		l.AddAtHead(val)
		return
    }
    
	node := NewNode(val)
	cur := l.Header
	for i := 0; i < index; i++ {
		cur = cur.Next
	}

	node.Next = cur.Next
	cur.Next = node
	l.Len++
}

// DeleteAtIndex 删除指定位置节点
func (l *LinkedList) DeleteAtIndex(index int) {
	if index > l.Len-1 || index < 0 {
		return
	}

	cur := l.Header
	for i := 0; i < index; i++ {
		cur = cur.Next
	}

	cur.Next = cur.Next.Next
	l.Len--
}

func (l *LinkedList) Scan() {
	cur := l.Header
	fmt.Println("---------")
	for cur != nil {
		fmt.Println(cur.Val)
		cur = cur.Next
	}
}

func main() {
	list := Constructor()
	list.AddAtHead(1)
	list.AddAtTail(3)	
	list.AddAtIndex(1, 2) //链表变为1-> 2-> 3
	list.Scan()
	list.Get(1) //返回2
	list.DeleteAtIndex(1) //现在链表是1-> 3
	list.Scan()
	list.Get(1) //返回3
}
~~~
{% enddetails %}

{% details Python code %}
~~~python
class Node:
    def __init__(self, val=-1):
        self.val = val
        self.next = None 

class MyLinkedList:
    def __init__(self):
        self.header = Node() 
        self.len = 0 

    def get(self, index: int) -> int:
        if index < 0 or index > self.len-1:
            return -1 
        cur = self.header 
        for _ in range(index+1):
            cur = cur.next 
        return cur.val

    def addAtHead(self, val: int) -> None:
        node = Node(val) 
        node.next = self.header.next 
        self.header.next = node 
        self.len += 1

    def addAtTail(self, val: int) -> None:
        cur = self.header 
        while cur.next:
            cur = cur.next 

        node = Node(val) 
        cur.next = node 
        self.len += 1 

    def addAtIndex(self, index: int, val: int) -> None:
        if index > self.len:
            return

        if index == self.len:
            self.addAtTail(val)
            return 

        if index < 0:
            self.addAtHead(val)     
            return

        cur = self.header 
        for _ in range(index):
            cur = cur.next 
        node = Node(val) 
        node.next = cur.next 
        cur.next = node 
        self.len += 1

    def deleteAtIndex(self, index: int) -> None:
        if index > self.len-1 or index < 0:
            return 

        cur = self.header 
        for _ in range(index):
            cur = cur.next 
        cur.next = cur.next.next
        self.len -= 1
~~~
{% enddetails %}

{% details 双向链表实现--设置虚拟头尾节点 %}
~~~go
type Node struct {
    Val int 
    Prev, Next *Node
}

type MyLinkedList struct {
    Head, Tail *Node 
    Len int 
}

// 创建一个有虚拟首尾节点的链表
func Constructor() MyLinkedList {
    head, tail := new(Node), new(Node)
    head.Next = tail 
    tail.Prev = head 
    return MyLinkedList{Head: head, Tail: tail, Len: 0}
}


func (l *MyLinkedList) Get(index int) int {
    if index < 0 || index >= l.Len {
        return -1
    }
    cur := l.Head
    for i := 0; i <= index; i++ {
        cur = cur.Next
    }
    return cur.Val 
}


func (l *MyLinkedList) AddAtHead(val int)  {
    // 直接在dummyHead后面添加一个节点即可
    // 后插法
    node := &Node{Val: val} 
    node.Next = l.Head.Next 
    node.Prev = l.Head 
    l.Head.Next.Prev = node 
    l.Head.Next = node 
    l.Len++
}


func (l *MyLinkedList) AddAtTail(val int)  {
    // 在DummyTail前面添加一个节点
    // 前插法添加节点
    node := &Node{Val: val} 
    node.Next = l.Tail 
    node.Prev = l.Tail.Prev 
    l.Tail.Prev.Next = node 
    l.Tail.Prev = node
    l.Len++
}


func (l *MyLinkedList) AddAtIndex(index int, val int)  {
    if index > l.Len {
        return
    }
    if index == l.Len {
        l.AddAtTail(val)
        return
    }
    if index < 0 {
        l.AddAtHead(val)
        return
    }

    cur := l.Head 
    for i := 0; i < index; i++ {
        cur = cur.Next
    }
    // 后插法
    node := &Node{Val: val} 
    node.Next = cur.Next 
    node.Prev = cur
    cur.Next.Prev = node 
    cur.Next = node 
    l.Len++
}


func (l *MyLinkedList) DeleteAtIndex(index int)  {
    if index < 0 || index >= l.Len {
        return
    }
    cur := l.Head 
    for i := 0; i < index; i++ {
        cur = cur.Next
    }
    cur.Next.Next.Prev = cur 
    cur.Next = cur.Next.Next
    l.Len--
}
~~~
{% enddetails %}


## 翻转链表
### [206. 反转链表](https://leetcode.cn/problems/reverse-linked-list/description/)
{% details 1.迭代法 %}
~~~go
func reverseList(head *ListNode) *ListNode {
    if head == nil || head.Next == nil {
        return head
    }

    // 申请节点
    var pre *ListNode
    // 记录下一节点
    var next *ListNode
    // 初始化当前节点
    cur := head
    for cur != nil {
        // 1: 先把下一轮的循环变量保存一下，为了第 3 步方便
        next = cur.Next
        // 第 2 步：实现当前节点的 next 指针的反转
        cur.Next = pre
        // 3: (往前移动)更新下一轮迭代的循环变量
        pre = cur
        cur = next
    }
    // 遍历完成以后，原来的最后一个节点就成为了 pre
    return pre
}
~~~
{% enddetails %}

{% details 2.递归法 %}
~~~go
func reverseList(head *ListNode) *ListNode {
    if head == nil || head.Next == nil {
        return head
    }
		// 只有当前节点和下一节点不为空，才能反转
    root := reverseList(head.Next)
    head.Next.Next = head
    head.Next = nil

    return root
}
~~~
{% enddetails %}

反转链表的性质
反转结束后，从原链表上看：
![反转链表的性质](../../assets/img/note_img/20230226202847.png){: width="500" height="300" }
- pre 指向反转这一段的末尾
- cur 指向反转这一段后续的下一个节点

### [92. 反转链表 II](https://leetcode.cn/problems/reverse-linked-list-ii/description/)
请你反转从位置 left 到位置 right 的链表节点，返回 反转后的链表 。 
> 思路：dummy Head + 反转链表性质

![](../../assets/img/note_img/20230226210959.png){: width="400" height="300" }  

若left=1,  此时是没有p0的，所以应加上一个哨兵节点
![](../../assets/img/note_img/20230226211229.png){: width="400" height="300" }  

{% details Golang code %}
~~~go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func reverseBetween(head *ListNode, left int, right int) *ListNode {
    dummy := &ListNode{Next: head} 
    p0 := dummy 
    for i := 0; i < left-1; i++ {
        p0 = p0.Next 
    }

    var pre *ListNode 
    cur := p0.Next 
    for i := 0; i < right-left+1; i++ {
        nxt := cur.Next 
        cur.Next = pre 
        pre = cur 
        cur = nxt 
    }
    p0.Next.Next = cur 
    p0.Next = pre 

    return dummy.Next
}
~~~
{% enddetails %}

{% details Python code %}
~~~python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def reverseBetween(self, head: Optional[ListNode], left: int, right: int) -> Optional[ListNode]:
        dummy = ListNode(next=head)
        p0 = dummy
        # 将p0移动到反转这一段的上一个节点
        for _ in range(left-1):
            p0 = p0.next

        # 然后就和反转链表一样的操作了
        pre = None
        cur = p0.next
        for _ in range(right-left+1):  # 反转这一段的长度为r-l+1
            nxt = cur.next
            cur.next = pre 
            pre = cur 
            cur = nxt

            # 拼接
        p0.next.next = cur 
        p0.next = pre 

        return dummy.next
~~~
{% enddetails %}

### [25. K 个一组翻转链表](https://leetcode.cn/problems/reverse-nodes-in-k-group/)

给你链表的头节点 head ，每 k 个节点一组进行翻转，请你返回修改后的链表。

思路：
- 判断 K 是否大于等于链表长度，不满足就不能反转, 反转过程同206
- p0 更新为下一段待反转链表的上一个节点，用临时变量保存nxt=p0.next
- 不断循环上述过程，得到结果
- 返回哨兵节点的next节点作为头节点
![](../../assets/img/note_img/20230226211029.png){: width="400" height="300" }

{% details Python code %}
~~~python
class Solution:
    def reverseKGroup(self, head: Optional[ListNode], k: int) -> Optional[ListNode]:
        n = 0 
        cur = head 
        while cur:
            n += 1
            cur = cur.next

        dummy = ListNode(next=head)
        p0 = dummy
        pre = None
        cur = p0.next
        while n >= k:
            n -= k  
            for _ in range(k):
                nxt = cur.next
                cur.next = pre 
                pre = cur 
                cur = nxt

            nxt = p0.next
            p0.next.next = cur 
            p0.next = pre 
            p0 = nxt  # p0 更新为下段待反转list的前1个节点

        return dummy.next 

~~~
{% enddetails %}

### [24. 两两交换链表中的节点](https://leetcode.cn/problems/swap-nodes-in-pairs/description/) 


1.递归
- 因为递归是重复做一样的事情，所以从宏观上考虑，只用考虑某一步是怎么完成的
- 假设待交换的俩节点分别为head和next，next的应该接受上一级返回的子链表(参考第2步)。就相当于是一个含三个节点的链表交换前两个节点
  
{% details Golang code %}
~~~go
func swapPairs(head *ListNode) *ListNode {
    if head == nil || head.Next == nil {
        return head
    }
    // subRoot是head.next.next之后的结点两两交换后的头结点
    
    nxt := head.Next
    head.Next = swapPairs(nxt.Next) 
    nxt.Next = head

    return nxt
    // ------------------------------
    // Method2
    one := head
    two := one.next
    three := two.next
    // 反转
    two.next = one
    one.next = swapParis(three)

    return two
}


~~~
{% enddetails %}

2.迭代思路同上。
{% details Python code %}
~~~python
class Solution:
    def swapPairs(self, head: Optional[ListNode]) -> Optional[ListNode]:
        if not head or not head.next:
            return head

        dummy = ListNode(next=head)
        p0 = dummy 
        cur = p0.next 
        pre = None 
        while cur and cur.next:
            for _ in range(2):
                nxt = cur.next 
                cur.next = pre 
                pre = cur 
                cur = nxt 
            nxt = p0.next
            p0.next.next = cur 
            p0.next = pre
            p0 = nxt 

        return dummy.next
~~~
{% enddetails %}


## 快慢指针
## 删除节点
## 合并/排序/相加