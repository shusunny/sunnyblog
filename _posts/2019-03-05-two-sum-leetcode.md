---
layout: post
title: "Leetcode中Two Sum练习"
author: "sun"
categories: Golang
tags: [Golang, Leetcode]
---

很久没有更博了，手有点生，就先写一些刷Leetcode时遇到的问题和一些感悟吧。

---
## Two Sum

Given an array of integers, return indices of the two numbers such that they add up to a specific target.

You may assume that each input would have exactly one solution, and you may not use the same element twice.

Example:

```
Given nums = [2, 7, 11, 15], target = 9,

Because nums[0] + nums[1] = 2 + 7 = 9,
return [0, 1].
```

此题是LT中的第一题，难度并不大，想清楚了其中的逻辑就不难解出。下面是我第一次提交的答案：

```go
func twoSum(nums []int, target int) []int {
	var r []int
	for i1, _ := range nums {
		for i2 := i1 + 1; i2 < len(nums); i2++ {
			if nums[i1]+nums[i2] == target {
				r = []int{i1, i2}
			}
		}
	}
	return r
}
```

虽然解决了问题的，但是这个算法排在了94%的位置。意味着此算法的效率极低，于是在思考了一番后，对算法进行了优化。

```go
func twoSum(nums []int, target int) []int {
	for i1, _ := range nums {
		for i2 := i1 + 1; i2 < len(nums); i2++ {
			if nums[i1]+nums[i2] == target {
				return []int{i1, i2}
			}
		}
	}
	return nil
}
```

在我们取消变量，并直接返回数组后，现在的算法排到了65%。有一些进步，但仍有很大的进步空间。于是在查看了discuss后，受启发用了map作了第三次提交。

```go
func twoSum(nums []int, target int) []int {
    m := make(map[int]int)
    for i, num := range nums{
        if v, ok := m[num]; ok{
            return []int{v, i}
        }
        m[target - num] = i
    }
    return nil
}
```

在我连连感叹此算法的精妙时，提交后发现用时为8ms，前面还有4ms的大神级算法。不禁让我点开瞻仰

---
## sample 4 ms submission

```go
import "sort"
type numberObj struct {
	num   int
	index int
}
type numberObjSlice []numberObj

func (n numberObjSlice) Len() int {
	return len(n)
}
func (n numberObjSlice) Less(i, j int) bool {
	return n[i].num < n[j].num
}
func (n numberObjSlice) Swap(i, j int) {
	n[i], n[j] = n[j], n[i]
}
func twoSumTemp(nums []numberObj, target int) []int {
	i := 0
	j := len(nums) - 1
	for i < j {
		sum := nums[i].num + nums[j].num
		if sum == target {
			return []int{nums[i].index, nums[j].index}
		} else if sum > target {
			j--
		} else {
			i++
		}
	}
	return []int{}
}
func twoSum(nums []int, target int) []int {
    b:=[]numberObj{}
    for i := 0; i < len(nums); i++ {
		b = append(b, numberObj{index: i, num: nums[i]})
	}
    sort.Sort(numberObjSlice(b))
    return twoSumTemp(b,target)
}
```

---

所以说算法这件事，精益求精，永远有更精简，更高效的算法等你去摸索。此问题虽然很简单，却很能看出一个人的算法功力。特别是对于那些没有系统性学习过数据结构和相关算法的同学们，在刷Leetcode时会显得尤其吃力。但是万丈高楼平地起，只要肯努力肯坚持，一定会得到满意的结果的。望与诸君共勉！