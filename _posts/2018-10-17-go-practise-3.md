---
layout: post
title: "Go语言练习 - Method"
author: "sun"
categories: Golang
tags: [Golang, Go Practise]
---

Go语言中方法(method)就是一类带特殊的"receiver"参数的函数。函数(func)的一般写法为
```
func (r receiver) identifier(parameters) (return(s)) { code }
```
方法和函数的区别就在于方法的receiver参数写在函数体外面，并且方法针对的对象是自定义的type类。不能为内建类型声明方法（如 int 等）

---
## 练习：Stringer

> 通过让 IPAddr 类型实现 fmt.Stringer 来打印点号分隔的地址。
例如，IPAddr{1, 2, 3, 4} 应当打印为 "1.2.3.4"。

原题模版代码为
```go
package main

import "fmt"

type IPAddr [4]byte

//TODO: Add a "String() string" method to IPAddr.

func main() {
	hosts := map[string]IPAddr{
		"loopback":  {127, 0, 0, 1},
		"googleDNS": {8, 8, 8, 8},
	}
	for name, ip := range hosts {
		fmt.Printf("%v: %v\n", name, ip)
	}
}

```
我们直接运行可以得到结果
```
loopback: [127 0 0 1]
googleDNS: [8 8 8 8]
```
题目要求我们为type IPAddr类添加一个方法，使IP地址的打印结果形如"127.0.0.1"而不是[127 0 0 1]

因此，我们需要在原题代码中加入下面的方法
```go
func (ip IPAddr) String() string {
	return fmt.Sprintf("%v.%v.%v.%v", ip[0], ip[1], ip[2], ip[3])
}
```
就可以得到所要求的输出结果：
```
googleDNS: 8.8.8.8
loopback: 127.0.0.1
```
---

## 练习：错误

> 从之前的练习中复制 Sqrt 函数，修改它使其返回 error 值。
Sqrt 接受到一个负数时，应当返回一个非 nil 的错误值。复数同样也不被支持。

> 创建一个新的类型
```go
type ErrNegativeSqrt float64
```
并为其实现
```go
func (e ErrNegativeSqrt) Error() string
方法使其拥有 error 值，通过 ErrNegativeSqrt(-2).Error() 调用该方法应返回 "cannot Sqrt negative number: -2"。
```
> *注意：* 在 Error 方法内调用 fmt.Sprint(e) 会让程序陷入死循环。可以通过先转换 e 来避免这个问题：fmt.Sprint(float64(e))。这是为什么呢？

> 修改 Sqrt 函数，使其接受一个负数时，返回 ErrNegativeSqrt 值。

根据题目，我们需要加入type ErrNegativeSqrt float64, 并为其添加方法使输入为负数时，打印"cannot Sqrt negative number: -2"。
所以我们可以对代码进行以下修改：
```go
package main

import (
	"fmt"
)

type ErrNegativeSqrt float64

func (e ErrNegativeSqrt) Error() string {
	return fmt.Sprintf("cannot Sqrt negative number: %g", e)
}

func Sqrt(x float64) (float64, error) {
	if x < 0 {
		return 0, ErrNegativeSqrt(x)
	} //引入错误处理的方法
	z := x
	for i := 1; i < 10; i++ {
		z -= (z*z - x) / (2 * z)
	} //这里我只用了简单的10次重复计算，也可以用第一次练习中的任何牛顿法函数
	return z, nil
}

func main() {
	fmt.Println(Sqrt(2))
	fmt.Println(Sqrt(-2))
}
```
可以得到以下运行结果：
```
1.4142135623730951 <nil>
0 cannot Sqrt negative number: -2
```
---

## 练习：Reader

> 实现一个 Reader 类型，它产生一个 ASCII 字符 'A' 的无限流。

题目给出如下代码
```go
package main

import "golang.org/x/tour/reader"

type MyReader struct{}

// TODO: Add a Read([]byte) (int, error) method to MyReader.

func main() {
	reader.Validate(MyReader{})
}
```
我们要做的就是添加产生很多个'A'的Read方法。向代码中加入如下方法：

```go
func (r MyReader) Read(b []byte) (int, error) {
	for i := range b {
		b[i] = 'A'
	}
	return len(b), nil
}
```
看到输出结果为"OK!"就OK啦。（其实刚开始我也没太看懂题目要干什么，在查看了加载包中的函数后才明白。
打开[golang.org/x/tour/reader](https://godoc.org/golang.org/x/tour/reader), 
再点击里面的[func Validate](https://github.com/golang/tour/blob/master/reader/validate.go)就能看到这个给我们输出OK的函数在后台做了些什么
