---
title: "Leetcode 链表篇 [二]"
date: 2022-11-03T22:26:27+08:00
draft: false
author: "chimission"
categories: ["algorithm"]
tags: ["leetcode"]
archives: ['2022', '2022-11']
comments: false
description: ""
url: "/posts/leetcode_link_2/"
---
>做题的一些笔记 链表篇

### [链表相交](https://leetcode.cn/problems/intersection-of-two-linked-lists-lcci/description/)  
由于前面几题的训练，这里自然而然的想到了快慢指针法。  
```go
func getIntersectionNode(headA, headB *ListNode) *ListNode {
    if headA == nil || headB == nil {
        return nil
    }
    //得到A的长度
    al := getLinkLenght(headA)
    //得到B的长度
    bl := getLinkLenght(headB)
    // 将长的链表提前移动到相同长度的位置
    if al>bl{
        for al>bl {
            headA = headA.Next
            al-=1
        }
    } 
    if bl>al {
        for bl>al {
            headB = headB.Next
            bl-=1
        }
    }
    //找到相等的节点
    for headA != headB{
            headA = headA.Next
            headB = headB.Next
    }
    return headA
    
}
func getLinkLenght(head *ListNode) int {
    l := 0
    for head.Next != nil {
        l++
        head = head.Next
    }
    return l
}
```