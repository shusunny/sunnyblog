---
layout: post
title: "Go语言练习 - 二叉查找树"
author: "sun"
categories: Golang
tags: [Golang, Go Practise]
---

## 练习：等价二叉查找树

题目过长，在此就不摘录了。可以查看[练习：等价二叉查找树](https://tour.go-zh.org/concurrency/7)。
要解答本题，我们首先需要查看Tree的[有关文档](https://godoc.org/golang.org/x/tour/tree#Tree)。
我们可以看到Tree的结构为
```go
type Tree struct {
    Left  *Tree
    Value int
    Right *Tree
}
```
并且Tree结构还带有一个生成树的方法
```go
func New(k int) *Tree
```
有了这些我们就可以先开始写我们的Walk函数
```go
// Walk 步进 tree t 将所有的值从 tree 发送到 channel ch。
func Walk(t *tree.Tree, ch chan int) {
	if t.Left != nil {
		Walk(t.Left, ch)
	}
	ch <- t.Value
	if t.Right != nil {
		Walk(t.Right, ch)
	}
	return
	//close(ch)
}
```
然后我们在主函数（func main）中打印出我们信道中的值
```go
func main() {
	ch := make(chan int)
	t1 := tree.New(1)

	//从信道中打印10个值
	go Walk(t1, ch)
	for i := 0; i < 10; i++ {
		fmt.Println(<-ch)
	}
}
```
就可以看到我们的程序输出了`1, 2, 3, ..., 10`。

可以注意到我们的Walk函数采用了return而没有用close(ch)来关闭信道，是因为Goroutine(`go Walk(t1, ch)`)是并发的，
在我们没有添加WaitGroup包前，打印函数for loop没有办法将值完全打印出来，即：可能刚从信道中取出1，2后，程序就关闭了。

---
下面我们写用来比较两个树是否相同的Same函数：
```go
// Same 检测树 t1 和 t2 是否含有相同的值。
func Same(t1, t2 *tree.Tree) bool {
	ch1, ch2 := make(chan int), make(chan int)
	go Walk(t1, ch1)
	go Walk(t2, ch2)
	if <-ch1 == <-ch2 {
		return true
	} else {
		return false
	}
}
```
再在我们的主函数中，对生成的tree.New(1), tree.New(2)进行比较
```go
func main() {
	t1 := tree.New(1)
	t2 := tree.New(2)
	//对tree1和tree2进行比较
	fmt.Println("tree 1 == tree 1:", Same(t1, t1))
	fmt.Println("tree 1 == tree 2:", Same(t1, t2))
}
```
得到的结果为
```go
tree 1 == tree 1: true
tree 1 == tree 2: false
```
这里我们只用了上面两个函数就完成了两个二叉树的比较，并且偷懒省略了WaitGroup和error的处理。
虽然在Go指南的环境中可以得到我们需要的结果，但程序本身是否稳定还有待检验。如果使用中出现了问题可以通过右上角的图标联系我。

根据我的经验，程序的并发可以提高程序的运行速度，但需要更强的逻辑性来保证程序以及数据的完整性，并应该及时适当地关闭并发程序，以免内存以及性能的浪费。

---
下面是本练习的完整程序:
```go
package main

import (
	"fmt"

	"golang.org/x/tour/tree"
)

// Walk 步进 tree t 将所有的值从 tree 发送到 channel ch。
func Walk(t *tree.Tree, ch chan int) {
	if t.Left != nil {
		Walk(t.Left, ch)
	}
	ch <- t.Value
	if t.Right != nil {
		Walk(t.Right, ch)
	}
	return
	//close(ch)
}

// Same 检测树 t1 和 t2 是否含有相同的值。
func Same(t1, t2 *tree.Tree) bool {
	ch1, ch2 := make(chan int), make(chan int)
	go Walk(t1, ch1)
	go Walk(t2, ch2)
	if <-ch1 == <-ch2 {
		return true
	} else {
		return false
	}
}

func main() {
	ch := make(chan int)
	t1 := tree.New(1)
	t2 := tree.New(2)

	//从信道中打印10个值
	go Walk(t1, ch)
	for i := 0; i < 10; i++ {
		fmt.Println(<-ch)
	}

	//对tree1和tree2进行比较
	fmt.Println("tree 1 == tree 1:", Same(t1, t1))
	fmt.Println("tree 1 == tree 2:", Same(t1, t2))
}
```
