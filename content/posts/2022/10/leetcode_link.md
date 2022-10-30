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