---
title: "Leetcode 字符串篇"
date: 2022-11-13T22:19:49+08:00
draft: false
author: "chimission"
categories: ["algorithm"]
tags: ["leetcode"]
archives: ['2022', '2022-11']
comments: false
description: ""
url: "/posts/leetcode_string_1/"
---
>做题的一些笔记 字符串篇
 <!--more-->

 ### [541. 反转字符串 II](https://leetcode.cn/problems/reverse-string-ii/)
 第一遍单纯的模拟操作，导致代码写的很冗余，设计了很多计数器
 ```go
 func reverseStr(s string, k int) string {
    l := len(s)
    if l <=1 || k==0{
        return s
    }
    sByte := []byte(s) 
    flag := true
    for i:=0; i<l;i++{
        if i % k==0 {
            flag = true
        }
        if flag {
            start:=i
            end:=i+k-1
            if end > l-1 {
                end = l-1
            }
            for start < end {
                sByte[start] , sByte[end] = sByte[end], sByte[start]
                start += 1
                end -= 1
        }
        i=i+k
        flag = false  
    }
    }
    return string(sByte)
}
 ```
 第二次看题解， 发现在循环的时候每次可以多循环几个元素，不用只加一  
 ```go
 func reverseStr(s string, k int) string {
    sByte := []byte(s)
    l := len(s)
    for i :=0; i<l;i+= 2 * k{
        start := i
        end := i+k-1
        if end>l-1{
            end=l-1
        }
        for start < end {
            sByte[start], sByte[end] = sByte[end], sByte[start]
            start+=1
            end-=1
        }

    }
    return string(sByte)
}
 ```