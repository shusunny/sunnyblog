---
layout: post
title: "Go语言练习 - Web 爬虫"
author: "sun"
categories: Golang
tags: [Golang, Go Practise]
---

## 练习：Web 爬虫

在这个练习中，我们将会使用 Go 的并发特性来并行化一个 Web 爬虫。

修改 Crawl 函数来并行地抓取 URL，并且保证不重复。

提示：你可以用一个 map 来缓存已经获取的 URL，但是要注意 map 本身并不是并发安全的！

---

我们需要做的是修改Crawl函数，使函数能够并行的，且不重复地抓取页面。

本题作为教程最后的压轴练习，需要对go语言有较深刻的了解，并熟练地掌握前几章所学的内容。下面是我的解答代码
```go
package main

import (
	"fmt"
	"sync"
)

type Fetcher interface {
	// Fetch 返回 URL 的 body 内容，并且将在这个页面上找到的 URL 放到一个 slice 中。
	Fetch(url string) (body string, urls []string, err error)
}

// -------------下面是我进行了修改的部分------------

// 创建map用来存放已抓取的网页, wg组用来管理线程
var m map[string]bool
var wg sync.WaitGroup

// TODO: 并行的抓取 URL。
// TODO: 不重复抓取页面。
// 这里用了辅助函数_crawl来完成上面的任务，并进行主要的抓取工作
func _crawl(url string, depth int, fetcher Fetcher, Results chan string, wg *sync.WaitGroup) {
	defer wg.Done()
	if depth <= 0 {
		return
	}

	if exists := m[url]; exists {
		return
	}

	body, urls, err := fetcher.Fetch(url)

	if err != nil {
		Results <- fmt.Sprintf("not found: %s", url)
		return
	}
	m[url] = true

	Results <- fmt.Sprintf("found: %s %q", url, body)

	for _, u := range urls {
		wg.Add(1)
		go _crawl(u, depth-1, fetcher, Results, wg)
	}
	return
}

// Crawl 使用 fetcher 从某个 URL 开始递归的爬取页面，直到达到最大深度。
// 这里Crawl的主要作用是管理线程，并从信道中读取数据
func Crawl(url string, depth int, fetcher Fetcher) {
	Results := make(chan string)
	wg.Add(1)
	go _crawl(url, depth, fetcher, Results, &wg)
	go func() {
		wg.Wait()
		close(Results)
	}()
	for i := range Results {
		fmt.Println(i)
	}
}

func main() {
	m = make(map[string]bool)
	Crawl("https://golang.org/", 4, fetcher)
}

// -------------下面是练习本身的代码--------------

// fakeFetcher 是返回若干结果的 Fetcher。
type fakeFetcher map[string]*fakeResult

type fakeResult struct {
	body string
	urls []string
}

func (f fakeFetcher) Fetch(url string) (string, []string, error) {
	if res, ok := f[url]; ok {
		return res.body, res.urls, nil
	}
	return "", nil, fmt.Errorf("not found: %s", url)
}

// fetcher 是填充后的 fakeFetcher。
var fetcher = fakeFetcher{
	"https://golang.org/": &fakeResult{
		"The Go Programming Language",
		[]string{
			"https://golang.org/pkg/",
			"https://golang.org/cmd/",
		},
	},
	"https://golang.org/pkg/": &fakeResult{
		"Packages",
		[]string{
			"https://golang.org/",
			"https://golang.org/cmd/",
			"https://golang.org/pkg/fmt/",
			"https://golang.org/pkg/os/",
		},
	},
	"https://golang.org/pkg/fmt/": &fakeResult{
		"Package fmt",
		[]string{
			"https://golang.org/",
			"https://golang.org/pkg/",
		},
	},
	"https://golang.org/pkg/os/": &fakeResult{
		"Package os",
		[]string{
			"https://golang.org/",
			"https://golang.org/pkg/",
		},
	},
}
```

运行程序可以得到结果
```go
found: https://golang.org/ "The Go Programming Language"
not found: https://golang.org/cmd/
found: https://golang.org/pkg/ "Packages"
found: https://golang.org/pkg/os/ "Package os"
not found: https://golang.org/cmd/
found: https://golang.org/pkg/fmt/ "Package fmt"
```

练习本身具有一定的难度，需要对已经给出的type类及相关方法有所了解，才好修改Crawl函数。
题目给出的模版函数可以给我们带来一定的思考和启发，需要先看懂题目自带的函数和方法以及相关变量。
（当然大神可以直接开写！但我相信很难得会有大神会来看我的blog）

至此Go Tour练习结束。以后我可能会发些别的有意思的golang练习和代码。我Go Tour的所有练习代码都可以在我的[Github Repo Go-tour-solutions](https://github.com/shusunny/Go-Practice/tree/master/Go-tour-solutions)中找到。

这里我还找到了Go Tour官方Github项目中给出的[解答代码](https://github.com/golang/tour/blob/master/solutions/webcrawler.go)，
可以打印出详细的抓取过程，并用了sync.Mutex锁保证读取的唯一性。我在此粘贴下来，以供参考

```go
// Copyright 2012 The Go Authors.  All rights reserved.
// Use of this source code is governed by a BSD-style
// license that can be found in the LICENSE file.

// +build ignore

package main

import (
	"errors"
	"fmt"
	"sync"
)

type Fetcher interface {
	// Fetch returns the body of URL and
	// a slice of URLs found on that page.
	Fetch(url string) (body string, urls []string, err error)
}

// fetched tracks URLs that have been (or are being) fetched.
// The lock must be held while reading from or writing to the map.
// See https://golang.org/ref/spec#Struct_types section on embedded types.
var fetched = struct {
	m map[string]error
	sync.Mutex
}{m: make(map[string]error)}

var loading = errors.New("url load in progress") // sentinel value

// Crawl uses fetcher to recursively crawl
// pages starting with url, to a maximum of depth.
func Crawl(url string, depth int, fetcher Fetcher) {
	if depth <= 0 {
		fmt.Printf("<- Done with %v, depth 0.\n", url)
		return
	}

	fetched.Lock()
	if _, ok := fetched.m[url]; ok {
		fetched.Unlock()
		fmt.Printf("<- Done with %v, already fetched.\n", url)
		return
	}
	// We mark the url to be loading to avoid others reloading it at the same time.
	fetched.m[url] = loading
	fetched.Unlock()

	// We load it concurrently.
	body, urls, err := fetcher.Fetch(url)

	// And update the status in a synced zone.
	fetched.Lock()
	fetched.m[url] = err
	fetched.Unlock()

	if err != nil {
		fmt.Printf("<- Error on %v: %v\n", url, err)
		return
	}
	fmt.Printf("Found: %s %q\n", url, body)
	done := make(chan bool)
	for i, u := range urls {
		fmt.Printf("-> Crawling child %v/%v of %v : %v.\n", i, len(urls), url, u)
		go func(url string) {
			Crawl(url, depth-1, fetcher)
			done <- true
		}(u)
	}
	for i, u := range urls {
		fmt.Printf("<- [%v] %v/%v Waiting for child %v.\n", url, i, len(urls), u)
		<-done
	}
	fmt.Printf("<- Done with %v\n", url)
}

func main() {
	Crawl("https://golang.org/", 4, fetcher)

	fmt.Println("Fetching stats\n--------------")
	for url, err := range fetched.m {
		if err != nil {
			fmt.Printf("%v failed: %v\n", url, err)
		} else {
			fmt.Printf("%v was fetched\n", url)
		}
	}
}

// fakeFetcher is Fetcher that returns canned results.
type fakeFetcher map[string]*fakeResult

type fakeResult struct {
	body string
	urls []string
}

func (f *fakeFetcher) Fetch(url string) (string, []string, error) {
	if res, ok := (*f)[url]; ok {
		return res.body, res.urls, nil
	}
	return "", nil, fmt.Errorf("not found: %s", url)
}

// fetcher is a populated fakeFetcher.
var fetcher = &fakeFetcher{
	"https://golang.org/": &fakeResult{
		"The Go Programming Language",
		[]string{
			"https://golang.org/pkg/",
			"https://golang.org/cmd/",
		},
	},
	"https://golang.org/pkg/": &fakeResult{
		"Packages",
		[]string{
			"https://golang.org/",
			"https://golang.org/cmd/",
			"https://golang.org/pkg/fmt/",
			"https://golang.org/pkg/os/",
		},
	},
	"https://golang.org/pkg/fmt/": &fakeResult{
		"Package fmt",
		[]string{
			"https://golang.org/",
			"https://golang.org/pkg/",
		},
	},
	"https://golang.org/pkg/os/": &fakeResult{
		"Package os",
		[]string{
			"https://golang.org/",
			"https://golang.org/pkg/",
		},
	},
}
```