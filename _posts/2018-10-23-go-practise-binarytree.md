---
layout: post
title: "Go语言练习 - 二叉查找树"
author: "sun"
categories: Golang
tags: [Golang, Go Practise]
---

## 练习：等价二叉查找树

1. 实现 Walk 函数。

2. 测试 Walk 函数。

函数 tree.New(k) 用于构造一个随机结构的已排序二叉查找树，它保存了值 k, 2k, 3k, ..., 10k。

创建一个新的信道 ch 并且对其进行步进：

go Walk(tree.New(1), ch)
然后从信道中读取并打印 10 个值。应当是数字 1, 2, 3, ..., 10。

3. 用 Walk 实现 Same 函数来检测 t1 和 t2 是否存储了相同的值。

4. 测试 Same 函数。

Same(tree.New(1), tree.New(1)) 应当返回 true，而 Same(tree.New(1), tree.New(2)) 应当返回 false。

Tree 的文档可在[这里](https://godoc.org/golang.org/x/tour/tree#Tree)找到。

要解答本题，我们首先需要查看Tree的有关文档。点开上面的链接，我们可以看到Tree的结构为
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
