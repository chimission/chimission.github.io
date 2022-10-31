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
	  -1

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