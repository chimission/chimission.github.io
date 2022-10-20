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
```go
func twoSum(nums []int, target int) []int {
    length := len(nums)
    r1, r2 := 0, 0 
    for i:=0; i<length;i++{
        for j:=i+1; j<length; j++ {
            if nums[i] + nums[j] == target {
                r1, r2 = i, j
            }
        }
    }
    return []int{r1,r2}
}
``` 

### [20.有效括号](https://leetcode.cn/problems/valid-parentheses/)
使用 hashmap 和 stack 做优化， 结束的if判断可以优化成一行，不用写n多行, 另外注意go 迭代string出来的的是byte
```go
func isValid(s string) bool {
	dict := map[byte]byte{'(': ')', '{': '}', '[': ']'}
	res := []byte{}
	for i := 0; i < len(s); i++ {
		if s[i] == '(' || s[i] == '{' || s[i] == '[' {
			res = append(res, dict[s[i]])
		} else {
			length := len(res)
			if length == 0 {
				return false
			}
			l := res[length-1]
			if l != s[i] {
				return false
			}
			res = res[:length-1]
		}
	}
	return len(res) == 0
}
```

### [704.二分查找](https://leetcode.cn/problems/binary-search/)
思维不难，主要考察代码编写能力，需要注意边界条件的判断。定义start和end， 然后用两者中元素和target对比，比target小就增大start，比target大就减小end，等于target就返回，一直循环如果 start比end大了就说明没有找到直接返回【未找到】
```go
func search(nums []int, target int) int {
	end := len(nums)
	start := 0
	mid := 0
	for start < end {
		mid = start + (end-start)/2
		if nums[mid] == target {
			return mid
		}
		if nums[mid] < target {
			start = mid + 1
		} else {
			end = mid
		}
	}
	return -1
}
```

### [27.移除元素](https://leetcode.cn/problems/remove-element/)
最开始暴力双循环解决， 看题解可以双指针， 有关多层循环的问题可以考虑多指针能不能优化时间。双指针法（快慢指针法）在数组和链表的操作中是非常常见的，很多考察数组、链表、字符串等操作的面试题，都使用双指针法。另外题目中可以改变元素顺序， 所以还有一种优化两个指针从首尾分贝向中间进行， 避免了多次重复的元素拷贝。普通双指针最坏情况2n， 首尾版双指针最坏n。  
```go
func removeElement(nums []int, val int) int {
	l := len(nums)
	e := 0
	for i := 0; i < l-e; {
		if nums[i] == val {
			e += 1
			for j := i + 1; j < l; j++ {
				nums[j-1] = nums[j]
			}
		} else {
			i += 1
		}
	}
	return l - e

}
```

### [977.有序数组的平方](https://leetcode.cn/problems/squares-of-a-sorted-array/submissions/)
又是一道双指针题目，需要在O(n)的时间里完成。暴力方法需要先平方再排序，时间O(n + nlog n)。好在前面做过了几道双指针，所以这道题也顺理成章的首先想到了双指针。在头尾各创建一个指针向中间靠拢，因为原数组本身是排好序的，所以指针元素平方本身有一定顺序，相当于省略了暴力中的排序过程。这道题需要注意的是我用的GO语言，如果提前声明了slice长度直接使用 `slice[index]=x` 来赋值结果就可以，之前写python多了产生了惯性思维，最开始声明了空slice，然后用的append头插插入结果，最后虽然也是O(n)的时间，但是因为使用了append的操作，实际上还是很耗时的。结果使用append 耗时300ms， 直接赋值耗时20ms，差距15倍。。。
```go
func sortedSquares(nums []int) []int {
	r := make([]int, len(nums))
	a := 0
	b, l := len(nums)-1, len(nums)-1
	for a <= b {
		a2 := nums[a] * nums[a]
		b2 := nums[b] * nums[b]
		if a2 >= b2 {
			r[l] = a2
			a++	
		} else {
			r[l] = b2
			b--
		}
    l--
	}
	return r

}
```

### [209.长度最小的子数组](https://leetcode.cn/problems/minimum-size-subarray-sum/submissions/)
首先最容想到的暴力双循环，但是人家不让这么做，最后一个用例数组很长会超时。
```go
func minSubArrayLen(target int, nums []int) int {
	l := len(nums)
	min_length := 0
	for first := 0; first < l; first++ {
		r := 0
		for secend := first; secend < l; {
			r += nums[secend]
			
			if r >= target {
				if (min_length == 0) || (secend-first+1 < min_length) {
					min_length = secend - first + 1
				}
                break
			} else if r < target {
				secend++
			}
		}

	}
	return min_length
}
```  
后来看了题解，用了`滑动窗口`的方法，其实也是双指针的变种，双循环是指定窗口头位置开始，双指针是从窗口尾位置开始，相当于跳过了很多双循环重复的结果，可以将 O(n2) 优化成 O(n)  
```go
func minSubArrayLen(target int, nums []int) int {
	min_length := len(nums) + 1
	sum := 0
	for first, second := 0, 0; first < len(nums); {
		sum += nums[first]
		for sum >= target {
			if first-second+1 < min_length {
				min_length = first - second + 1
			}
			sum -= nums[second]
			second++
		}
		first++
	}
	if min_length == len(nums)+1 {
		return 0
	}
	return min_length
}
```