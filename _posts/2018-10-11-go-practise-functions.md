---
layout: post
title: "Go语言练习 - 循环与函数"
author: "sun"
categories: Golang
tags: [Golang, Go Practise]
---

[A Tour of Go](https://tour.golang.org/)是官方做的一个很棒Golang学习文档。个人觉得很适合用来学习go语言的基本用法。但是其课程内容较少，跳跃性很大，感觉像前面还在学1+1，后面练习就让你独立解方程了。因此摘选了Tour上的部分习题做解答并作为个人练习。

## 练习：循环与函数

此练习位于流程与控制栏（control flow），原题如下：

> 为了练习函数与循环，我们来实现一个平方根函数：用牛顿法实现平方根函数。
计算机通常使用循环来计算 x 的平方根。从某个猜测的值 z 开始，我们可以根据 z² 与 x 的近似度来调整 z，产生一个更好的猜测：
```
z -= (z*z - x) / (2*z) 
```
> 重复调整的过程，猜测的结果会越来越精确，得到的答案也会尽可能接近实际的平方根。
在提供的 func Sqrt 中实现它。无论输入是什么，对 z 的一个恰当的猜测为 1。 要开始，请重复计算 10 次并随之打印每次的 z 值。观察对于不同的值 x（1、2、3 ...）， 你得到的答案是如何逼近结果的，猜测提升的速度有多快。
> 提示：用类型转换或浮点数语法来声明并初始化一个浮点数值：
```
z := 1.0
z := float64(1)
```
> 然后，修改循环条件，使得当值停止改变（或改变非常小）的时候退出循环。观察迭代次数大于还是小于 10。 尝试改变 z 的初始猜测，如 x 或 x/2。你的函数结果与标准库中的 math.Sqrt 接近吗？

> （注：如果你对该算法的细节感兴趣，上面的 z² − x 是 z² 到它所要到达的值（即 x）的距离，除以的 2z 为 z² 的导数，我们通过 z² 的变化速度来改变 z 的调整量。这种通用方法叫做牛顿法。它对很多函数，特别是平方根而言非常有效。）

此题主要练习

 - 浮点变量的声明，赋值
 - for循环
 - 主要函数的调用

我们现在做重复计算 10 次并随之打印每次的 z 值

```go
package main

import (
	"fmt"
)

func Sqrt(x float64) float64 {
	z := 1.0 //赋值浮点数
	for i := 1; i < 11; i++ {
		z -= (z*z - x) / (2 * z)
		fmt.Println(i) //打印次数
		fmt.Println(z) //打印当前z值
	}
	return z //返回最终值
}

func main() {
	fmt.Println(Sqrt(2))
}
```
运行程序可以得到以下结果
```
1
1.5
2
1.4166666666666667
3
1.4142156862745099
4
1.4142135623746899
5
1.4142135623730951
6
1.414213562373095
7
1.4142135623730951
8
1.414213562373095
9
1.4142135623730951
10
1.414213562373095
1.414213562373095
```
我们可以看到第5次迭代后已经得到了稳定结果。下面我们修改循环条件，使得当值停止改变（或改变非常小）的时候退出循环

```go
package main

import (
	"fmt"
	"math"
)

func Sqrt(x float64) float64 {
	z := 1.0
	temp := 0.0 // 临时变量，用来记录前一次z的值
	for {
		z = z - (z*z-x)/(2*z) // 计算出最新的z值
		fmt.Println(z)
		if math.Abs(z-temp) < 0.000000000000001 {
			break // 用math库中的绝对值使值停止改变（或改变非常小）的时候退出循环
		} else {
			temp = z // 改变仍较大时记录此时的z值
		}
	}
	return z
}

func main() {
	fmt.Println("Our Result:\t", Sqrt(2))
	fmt.Println("Accurate Result:", math.Sqrt(2))
}
```
运行这段程序，我们可以得到以下输出结果

```
1.5
1.4166666666666667
1.4142156862745099
1.4142135623746899
1.4142135623730951
1.414213562373095
Our Result:	 1.414213562373095
Accurate Result: 1.4142135623730951
```
可以看到我们的结果与标准数值一致（在float64的数值精确度下）。
这个应该是Go Tour中最简单的练习了，但对于毫无编程基础的同学还是有一些挑战的。

Anyway, 有任何问题都可以点右上角的图标联系我