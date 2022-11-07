---
title: "Leetcode 哈希篇"
date: 2022-11-03T22:26:27+08:00
draft: false
author: "chimission"
categories: ["algorithm"]
tags: ["leetcode"]
archives: ['2022', '2022-11']
comments: false
description: ""
url: "/posts/leetcode_hash_1/"
---
>做题的一些笔记 hash篇
 <!--more-->

 ### [454. 四数相加 II](https://leetcode.cn/problems/4sum-ii/comments/)  
 这道题虽然想到了hash可以降低复杂度，但是没有想到可以分组，AB一组，CD一组。题解将4层for循坏利用分组+hash拆解成两层for循环。hash表将AB和作为key，将AB和出现的次数作为value。wiki：在寻找和的次数类似的题目时就要考虑能不能用hash降低复杂度。
 ```go
 func fourSumCount(nums1 []int, nums2 []int, nums3 []int, nums4 []int) int {
    result:=0
    countAB := make(map[int]int)
    for _, a := range(nums1){
        for _, b := range(nums2) {
            countAB[a+b] += 1
        }
    }
    for _, c := range(nums3) {
        for _, d:= range(nums4){
            result += countAB[0-c-d]
        }
    }
    return result
}
 ```