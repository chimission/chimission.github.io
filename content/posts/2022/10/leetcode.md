---
title: "Leetcode 笔记"
date: 2022-10-08T22:09:46+08:00
draft: false
author: "chimission"
categories: ["algorithm"]
tags: ["leetcode"]
archives: ['2022', '2022-10']
comments: false
description: ""
url: "/posts/leetcode/"
---
>做题的一些笔记
 <!--more-->
### [1.两数之和](https://leetcode.cn/problems/valid-parentheses/)
求数组内两个元素特定和的下标， 刚开始我用两个 for 循环做，n平方时间复杂度。  
涉及到查找和计算两个方面就可以考虑使用 hashmap 优化，可以降低时间复杂度到n。  

### [20.有效括号](https://leetcode.cn/problems/valid-parentheses/)
使用 hashmap 和 stack 做优化， 结束的if判断可以优化成一行，不用写n多行, 另外注意go 迭代string出来的的是byte
```go
if len(stake) != 0 {
    return false        可以优化为       return len(stake)!=0
}
    return true
```

### [704.二分查找](https://leetcode.cn/problems/binary-search/)
思维不难，主要考察代码编写能力，需要注意边界条件的判断。定义start和end， 然后用两者中间元素和target对比，比target小就增大start，比target大就减小end，等于target就返回，一直循环如果 start比end大了就说明没有找到直接返回【未找到】

### [27.移除元素](https://leetcode.cn/problems/remove-element/)
最开始暴力双循环解决， 看题解可以双指针， 有关多层循环的问题可以考虑多指针能不能优化时间。双指针法（快慢指针法）在数组和链表的操作中是非常常见的，很多考察数组、链表、字符串等操作的面试题，都使用双指针法。另外题目中可以改变元素顺序， 所以还有一种优化两个指针从首尾分贝向中间进行， 避免了多次重复的元素拷贝。普通双指针最坏情况2n， 首尾版双指针最坏n。