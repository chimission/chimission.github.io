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
 <!--more-->
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
### [142. 环形链表 II](https://leetcode.cn/problems/linked-list-cycle-ii/description/)  
一道题考了两个地方，一个链表相交，一个求相交的节点。第一点比较简单，解决方法还是经典的快慢指针，但是第二点相交节点有困难，它并不是单纯的编程题，需要一点数学证明。我刚开始想到了用字典，但是感觉不太优雅还要引入额外的数据结构，当然如果是业务上遇到这种情况我肯定会用字典，毕竟怎么简单怎么来，但是做题嘛，还是想下最优解怎么写，最后搜了下 [题解](https://programmercarl.com/0142.%E7%8E%AF%E5%BD%A2%E9%93%BE%E8%A1%A8II.html#_142-环形链表ii)。
```go
func detectCycle(head *ListNode) *ListNode {
    fast:=head
    slow := head
    for fast!=nil && fast.Next!=nil {
        fast = fast.Next.Next
        slow = slow.Next
        if fast == slow {
            index1:=head
            index2:=slow
            for index1 != index2{
                index1 = index1.Next
                index2 = index2.Next
            }
            return index1
        }
    }
    return nil
}
```