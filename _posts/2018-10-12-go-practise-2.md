---
layout: post
title: "Go Tour 练习(2)"
author: "sun"
categories: Golang
tags: [Golang, Go Practise]
---

Go语言中的切片（Slice）是比较重要的内容，下面是一个有关切片的练习

## 二、练习：切片

此练习位于流程与控制栏（control flow），原题如下：

> 实现 Pic。它应当返回一个长度为 dy 的切片，其中每个元素是一个长度为 dx，元素类型为`uint8`的切片。
当你运行此程序时，它会将每个整数解释为灰度值（好吧，其实是蓝度值）并显示它所对应的图像。

> 图像的选择由你来定。几个有趣的函数包括 (x+y)/2, x*y, x^y, x*log(y) 和 x%(y+1)。
（提示：需要使用循环来分配 [][]uint8 中的每个 []uint8；请使用 uint8(intValue) 在类型之间转换；你可能会用到 math 包中的函数。）

此题主要练习

 - 切片的使用
 - for循环语句的嵌套

解答代码如下
```go
package main

import (
	"golang.org/x/tour/pic"
)

func Pic(dx, dy int) [][]uint8 {
	var ret = make([][]uint8, dy)
	//外层切片
	for x := 0; x < dy; x++ {
		ret[x] = make([]uint8, dx)
		//内层切片
		for y := 0; y < dx; y++ {
			ret[x][y] = uint8((x + y) / 2)
			//给内层切片每个元素赋值，可以更改为题目中的其他函数
		}
	}
	return ret
}

func main() {
	pic.Show(Pic)
}

```
运行程序可以得到256px*256px的图片，如下图所示

![]({{ site.github.url }}/assets/img/slice_solution.png "x+y/2图像")

有兴趣的同学还可以查看我们载入包[golang.org/x/tour/pic](https://godoc.org/golang.org/x/tour/pic)中的内容，
以及其中的[pic程序](https://github.com/golang/tour/blob/master/pic/pic.go)的有关代码，深入地了解上面的图片是如何生成的。
