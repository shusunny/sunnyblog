---
layout: post
title: "Go语言练习 - 切片和map"
author: "sun"
categories: Golang
tags: [Golang, Go Practise]
---

Go语言中的切片（Slice）是比较重要的内容，下面是一个有关切片的练习

## 二、练习：切片

> 实现 Pic。它应当返回一个长度为 dy 的切片，其中每个元素是一个长度为 dx，元素类型为`uint8`的切片。当你运行此程序时，它会将每个整数解释为灰度值（好吧，其实是蓝度值）并显示它所对应的图像。

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

有兴趣的同学还可以查看我们载入包[golang.org/x/tour/pic](https://godoc.org/golang.org/x/tour/pic)中的内容，以及其中的[pic程序](https://github.com/golang/tour/blob/master/pic/pic.go)的有关代码，深入地了解上面的图片是如何生成的。

---

## 三、练习：映射
> 实现 WordCount。它应当返回一个映射，其中包含字符串 s 中每个“单词”的个数。函数 wc.Test 会对此函数执行一系列测试用例，并输出成功还是失败。

> 你会发现 [strings.Fields](https://go-zh.org/pkg/strings/#Fields) 很有帮助。

题目主要练习for range的用法，和map类型。

我们可以先打开上面的链接看下strings.Fields的用法。strings.Fields主要将一段话中的各单词提取出来，如示例
```go
func main() {
	fmt.Printf("Fields are: %q", strings.Fields("  foo bar  baz   "))
}
```
语句将输出"foo","bar"和"baz", 所以我们用string.Fields(s)会把语句s中的单词列出。所以此练习的代码为
```go
package main

import (
	"strings"

	"golang.org/x/tour/wc"
)

func WordCount(s string) map[string]int {
	m := make(map[string]int) //创建map
	for _, c := range strings.Fields(s) {
		//遍历s，并对出现的每一个词汇在我们的计数器c上加1
		m[c]++
	}
	return m
}

func main() {
	wc.Test(WordCount)
}
```
程序运行输出结果如下：
```
PASS
 f("I am learning Go!") = 
  map[string]int{"learning":1, "Go!":1, "I":1, "am":1}
PASS
 f("The quick brown fox jumped over the lazy dog.") = 
  map[string]int{"The":1, "quick":1, "jumped":1, "over":1, "brown":1, "fox":1, "the":1, "lazy":1, "dog.":1}
PASS
 f("I ate a donut. Then I ate another donut.") = 
  map[string]int{"another":1, "I":2, "ate":2, "a":1, "donut.":2, "Then":1}
PASS
 f("A man a plan a canal panama.") = 
  map[string]int{"canal":1, "panama.":1, "A":1, "man":1, "a":2, "plan":1}
```
---

## 四、练习：斐波纳契闭包
> 实现一个 fibonacci 函数，它返回一个函数（闭包），该闭包返回一个斐波纳契数列 `(0, 1, 1, 2, 3, 5, ...)`。

题目主要练习函数的闭包。原题模版如下
```go
package main

import "fmt"

// fibonacci is a function that returns
// a function that returns an int.
func fibonacci() func() int {
}

func main() {
	f := fibonacci()
	for i := 0; i < 10; i++ {
		fmt.Println(f())
	}
}
```
可以看到已经有打印结果的主函数了，我们要做的只是将fibonacci数列的逻辑写入到func fibonacii中去。解答如下
```go
package main

import "fmt"

// fibonacci is a function that returns
// a function that returns an int.
func fibonacci() func() int {
	f, g := 1, 0 
	//因为下面加法运算完后才返回f，且最终打印值为f()也就是f的值。所以如果我们想返回从0开始的数列，需要将g初始赋值为0
	return func() int {
		f, g = g, f+g
		return f
	}
}

func main() {
	f := fibonacci()
	for i := 0; i < 10; i++ {
		fmt.Println(f())
	}
}
```
上面就是Go语言之旅中三个基础练习的解答
