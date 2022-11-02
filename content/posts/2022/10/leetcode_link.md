---
title: "Leetcode 链表篇"
date: 2022-10-27T22:10:57+08:00
draft: false
author: "chimission"
categories: [""]
tags: [""]
archives: ['2022', '2022-10']
comments: false
description: ""
url: "/posts/leetcode_link/"
---
>做题的一些笔记 链表篇
 <!--more-->
 ### [203. 移除链表元素](https://leetcode.cn/problems/remove-linked-list-elements/)
 ```go
 func removeElements(head *ListNode, val int) *ListNode {
	if head == nil {
		return head
	}
	t := head
	for t.Next != nil {
		if t.Next.Val == val {
			t.Next = t.Next.Next
		} else {
			t = t.Next
		}
	}
    if head.Val == val {
		head = head.Next
	}
	return head
}
 ```
 第一版自己写的代码感觉不太优雅， 特殊处理的代码块有两个， 第一个判断是不是空链表，第二个是循环完了,回过头来判断头节点是不是符合条件。看看能不能优化成一个for循环就可以，去掉这两个if判断。  
 看了题解，通常做法是在进入循环前在链表最前面加了个新的头结点，这样就不用循环完回过头来判断头结点的条件。  
 另外有人给出了递归的版本，这个我开始往递归上想，借鉴一下。  
 ```go
 func removeElements(head *ListNode, val int) *ListNode {
	if head == nil {
		return head
	}
	t := head
	for t.Next != nil {
		if t.Next.Val == val {
			t.Next = t.Next.Next
		} else {
			t = t.Next
		}
	}
    if head.Val == val {
		head = head.Next
	}
	return head
}
 ```  

### [707. 设计链表](https://leetcode.cn/problems/design-linked-list/submissions/)  
虽然题目标记是中等难度， 但是不太涉及算法技巧，真正考验写代码的能力，设计一个链表的crud。有些方法思维很简单，但是落实到代码上就不是那么回事了，涉及到各种边界的判断。最后还行，看提交记录19年用py3通过了，这次用Go再通过一次。:-)  
```go
type LinkedNode struct {
	Val  int
	Next *LinkedNode
}

type MyLinkedList struct {
	heade *LinkedNode
}

func Constructor() MyLinkedList {
	return *new(MyLinkedList)

}

func (this *MyLinkedList) Get(index int) int {

	t := this.heade
	currentIndex := 0
	for t != nil {
		if currentIndex == index {
			return t.Val
		}
		currentIndex += 1
		t = t.Next

	}
	return -1

}

func (this *MyLinkedList) AddAtHead(val int) {
	t := this.heade
	this.heade = &LinkedNode{Val: val, Next: t}
}

func (this *MyLinkedList) AddAtTail(val int) {
	t := this.heade
    if this.heade == nil {
		this.AddAtHead(val)
		return
	}
	for t.Next != nil {
		t = t.Next
	}
	t.Next = &LinkedNode{Val: val, Next: nil}
}

func (this *MyLinkedList) AddAtIndex(index int, val int) {
	if index <= 0 {
		this.AddAtHead(val)
	}
	t := this.heade
	currentIndex := 0
	for t != nil {
		if currentIndex == index-1 {
			pre := t.Next
			t.Next = &LinkedNode{Val: val, Next: pre}
		}
		currentIndex += 1
		t = t.Next
	}

}

func (this *MyLinkedList) DeleteAtIndex(index int) {
	if index == 0 {
		this.heade = this.heade.Next
		return
	}
	t := this.heade
	currentIndex := 0
	for t.Next != nil {
		if currentIndex == index-1 {
			t.Next = t.Next.Next
			return
		}
		currentIndex += 1
		t = t.Next
	}

}
/**
 * Your MyLinkedList object will be instantiated and called as such:
 * obj := Constructor();
 * param_1 := obj.Get(index);
 * obj.AddAtHead(val);
 * obj.AddAtTail(val);
 * obj.AddAtIndex(index,val);
 * obj.DeleteAtIndex(index);
 */
```

### [206. 反转链表](https://leetcode.cn/problems/reverse-linked-list/)
这道题给我打击很大，一上来思路很清晰，但是代码就是写不出来， 看了下自己的提交记录18年就通过了一次，19年也通过了一次，如今再来做一遍居然做不出来了。。
也没啥好说的，不涉及到思维技巧，就是考察写代码能力。还需要多多练习啊。。。
```go
//迭代法
func reverseList(head *ListNode) *ListNode {
    var p *ListNode
    c := head
    for c!=nil {
        t:=c.Next
        c.Next=p
        p=c
        c=t
    }
    return p
}
// 递归法
func reverseList(head *ListNode) *ListNode {
    if head!=nil && head.Next!=nil{
		p:=reverseList(head.Next)
		head.Next.Next=head
		head.Next=nil
		return p
	}
	return head
}
```  

### [24. 两两交换链表中的节点](https://leetcode.cn/problems/swap-nodes-in-pairs/)  
这道题有3种不懂思路的方法，记录一下
```go
// 迭代
func swapPairs(head *ListNode) *ListNode {
    d := &ListNode{0, head}
    cur := d
    for cur.Next!=nil && cur.Next.Next!=nil{
        s := cur.Next
        t := cur.Next.Next.Next

        cur.Next = cur.Next.Next
        cur.Next.Next = s
        cur.Next.Next.Next = t
        cur = cur.Next.Next
    }
    return d.Next
}
//递归
func swapPairs(head *ListNode) *ListNode {
    if head == nil || head.Next == nil {
		return head
	}
    nextItem := head.Next
	head.Next = swapPairs(nextItem.next)
	nextItem.Next = head
	return nextItem
}
// 栈
func swapPairs(head *ListNode) *ListNode {
    var linkList [2]*ListNode

    d := &ListNode{0, head}
	cur := head
    head =d
    for cur != nil && cur.Next != nil {
        linkList[0] = cur
		linkList[1] = cur.Next
        cur = cur.Next.Next
        d.Next = linkList[1]
        d.Next.Next = linkList[0]
        d = d.Next.Next
    }
    if cur != nil {
		d.Next = cur
	} else {
		d.Next = nil
	}
    return head.Next
}
```

### [19.删除链表的倒数第N个节点](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/)
题目是中等难度的题， 单纯的模拟操作并没有什么难度，先计算长度，再转换删除节点的位置，最后转化为删除第n个节点。用了两次循环，但也是O(n)。  
还有进阶版只一次循环，双指针法， 分别设置fast和slow两个指针，先让fast先走n步，然后fast和slow一起走，当fast走到底时，slow的下一个节点就是要删除的节点。
```go
//循环迭代
func removeNthFromEnd(head *ListNode, n int) *ListNode {
    // 先求链表长度
    length := 0
    for cur := head; cur!= nil;cur=cur.Next {
        length++
    }
    
    // 定位被删除的节点是第几个节点
    index := length - n
    // 头节点删除
    if index == 0 {
        return head.Next
    }
    //删除节点
    t := head
	currentIndex := 0
	for t.Next != nil {
		if currentIndex == index-1 {
			t.Next = t.Next.Next
			return head
		}
		currentIndex += 1
		t = t.Next
	}
    return head
}
// 双指针
func removeNthFromEnd(head *ListNode, n int) *ListNode {
    
    slow , fast := head, head
    
    for fast.Next != nil {
        if n > 0 {
        fast = fast.Next
        n--
        } else{
            fast = fast.Next
            slow = slow.Next
        }
    }
    if n>0 { // 说明是头节点需要删除
        return head.Next
    } else {
    slow.Next = slow.Next.Next
    }
    return head
}
//
```