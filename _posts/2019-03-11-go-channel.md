---
layout: post
title: "有关Go Channel"
author: "sun"
categories: Golang
tags: [Golang, Go Practise]
---

channel是Go语言在语言级别提供的goroutine间的通信方式。我们可以使用channel在两个或多个goroutine之间传递消息。
channel是进程内的通信方式，因此通过channel传递对象的过程和调用函数时的参数传递行为比较一致，比如也可以传递指针等。如果需要跨进程通信，我们建议用分布式系统的方法来解决，比如使用Socket或者HTTP等通信协议。Go语言对于网络方面也有非常完善的支持。

channel是类型相关的。也就是说，一个channel只能传递一种类型的值，这个类型需要在声明channel时指定。如果对Unix管道有所了解的话，就不难理解channel，可以将其认为是一种类型安全的管道。

1. 基本语法
一般channel的声明形式为：

`var chanName chan ElementType`

与一般的变量声明不同的地方仅仅是在类型之前加了chan关键字。 ElementType指定这个channel所能传递的元素类型。举个例子，我们声明一个传递类型为int的channel：

`var ch chan int`

或者，我们声明一个map，元素是bool型的channel:

`var m map[string] chan bool`

上面的语句都是合法的。

定义一个channel也很简单，直接使用内置的函数make()即可：

`ch := make(chan int)`

这就声明并初始化了一个int型的名为ch的channel。

在channel的用法中，最常见的包括写入和读出。将一个数据写入（发送）至channel的语法很直观，如下：

`ch <- value`

向channel写入数据通常会导致程序阻塞，直到有其他goroutine从这个channel中读取数据。从 channel中读取数据的语法是:

`value := <-ch`

如果channel之前没有写入数据，那么从channel中读取数据也会导致程序阻塞，直到channel中被写入数据为止。我们之后还会提到如何控制channel只接受写或者只允许读取，即单向channel。

2. select
早在Unix时代，select机制就已经被引入。通过调用select()函数来监控一系列的文件句柄，一旦其中一个文件句柄发生了IO动作，该select()调用就会被返回。后来该机制也被用于实现高并发的Socket服务器程序。

Go语言直接在语言级别支持select关键字，用于处理异步IO问题。

select的用法与switch语言非常类似，由select开始一个新的选择块，每个选择条件由case语句来描述。与switch语句可以选择任何可使用相等比较的条件相比， select有比较多的限制，其中最大的一条限制就是每个case语句里必须是一个IO操作，大致的结构如下：

```go
select {
    case <-chan1:
    // 如果chan1成功读到数据，则进行该case处理语句
    case chan2 <- 1:
    // 如果成功向chan2写入数据，则进行该case处理语句
    default:
    // 如果上面都没有成功，则进入default处理流程
}
```
可以看出， select不像switch，后面并不带判断条件，而是直接去查看case语句。每个case语句都必须是一个面向channel的操作。比如上面的例子中，第一个case试图从chan1读取一个数据并直接忽略读到的数据，而第二个case则是试图向chan2中写入一个整型数1，如果这两者都没有成功，则到达default语句。

基于此功能，我们可以实现一个有趣的程序：

```go
ch := make(chan int, 1)
for {
    select {
        case ch <- 0:
        case ch <- 1:
    }
    i := <-ch
    fmt.Println("Value received:", i)
}
```
这个程序实现了一个随机向ch中写入一个0或者1 的过程，这是个死循环。

3. 缓冲机制
之前的示例都是不带缓冲的channel，这种做法对于传递单个数据的场景可以接受，但对于需要持续传输大量数据的场景就有些不合适了。接下来我们介绍如何给channel带上缓冲，从而达到消息队列的效果。

要创建一个带缓冲的channel，其实也非常容易：
```go
c := make(chan int, 1024)
```
在调用make()时将缓冲区大小作为第二个参数传入即可，比如上面这个例子就创建了一个大小为1024的int类型channel，即使没有读取方，写入方也可以一直往channel里写入，在缓冲区被填完之前都不会阻塞。

从带缓冲的channel中读取数据可以使用与常规非缓冲channel完全一致的方法，但我们也可以使用range关键来实现更为简便的循环读取：
```go
for i := range c {
    fmt.Println("Received:", i)
}
```
4. 超时机制
在之前对channel的介绍中，我们完全没有提到错误处理的问题，而这个问题显然是不能被忽略的。在并发编程的通信过程中，最需要处理的就是超时问题，即向channel写数据时发现channel已满，或者从channel试图读取数据时发现channel为空。如果不正确处理这些情况，很可能会导致整个goroutine锁死。

虽然goroutine是Go语言引入的新概念，但通信锁死问题已经存在很长时间，在之前的C/C++开发中也存在。操作系统在提供此类系统级通信函数时也会考虑入超时场景，因此这些方法通常都会带一个独立的超时参数。超过设定的时间时，仍然没有处理完任务，则该方法会立即终止并返回对应的超时信息。超时机制本身虽然也会带来一些问题，比如在运行比较快的机器或者高速的网络上运行正常的程序，到了慢速的机器或者网络上运行就会出问题，从而出现结果不一致的现象，但从根本上来说，解决死锁问题的价值要远大于所带来的问题。

使用channel时需要小心，比如对于以下这个用法：
```go
i := <-ch
```
不出问题的话一切都正常运行。但如果出现了一个错误情况，即永远都没有人往ch里写数据，那么上述这个读取动作也将永远无法从ch中读取到数据， 导致的结果就是整个goroutine永远阻塞并没有挽回的机会。如果channel只是被同一个开发者使用，那样出问题的可能性还低一些。但如果一旦对外公开，就必须考虑到最差的情况并对程序进行保护。

Go语言没有提供直接的超时处理机制，但我们可以利用select机制。虽然select机制不是专为超时而设计的，却能很方便地解决超时问题。因为select的特点是只要其中一个case已经完成，程序就会继续往下执行，而不会考虑其他case的情况。

基于此特性，我们来为channel实现超时机制：

```go
// 首先，我们实现并执行一个匿名的超时等待函数
timeout := make(chan bool, 1)
go func() {
    time.Sleep(1e9) // 等待1秒钟
    timeout <- true
}()

// 然后我们把timeout这个channel利用起来
select {
    case <-ch:
    // 从ch中读取到数据
    case <-timeout:
    // 一直没有从ch中读取到数据，但从timeout中读取到了数据
}
```
这样使用select机制可以避免永久等待的问题，因为程序会在timeout中获取到一个数据后继续执行，无论对ch的读取是否还处于等待状态，从而达成1秒超时的效果。

这种写法看起来是一个小技巧，但却是在Go语言开发中避免channel通信超时的最有效方法。在实际的开发过程中，这种写法也需要被合理利用起来，从而有效地提高代码质量。

5. channel的传递
需要注意的是，在Go语言中channel本身也是一个原生类型，与map之类的类型地位一样，因此channel本身在定义后也可以通过channel来传递。

我们可以使用这个特性来实现Linux上非常常见的管道（pipe）特性。管道也是使用非常广泛的一种设计模式，比如在处理数据时，我们可以采用管道设计，这样可以比较容易以插件的方式增加数据的处理流程。

下面我们利用channel可被传递的特性来实现我们的管道。为了简化表达，我们假设在管道中传递的数据只是一个整型数，在实际的应用场景中这通常会是一个数据块。

首先限定基本的数据结构：
```go
type PipeData struct {
    value int
    handler func(int) int
    next chan int
}
```
然后我们写一个常规的处理函数。我们只要定义一系列PipeData的数据结构并一起传递给这个函数，就可以达到流式处理数据的目的：
```go
func handle(queue chan *PipeData) {
    for data := range queue {
        data.next <- data.handler(data.value)
    }
}
```
这里我们只给出了大概的样子，同理，利用channel的这个可传递特性，我们可以实现非常强大、灵活的系统架构。与Go语言接口的非侵入式类似，channel的这些特性也可以大大降低开发者的心智成本，用一些比较简单却实用的方式来达成在其他语言中需要使用众多技巧才能达成的效果。

6. 单向channel
顾名思义，单向channel只能用于发送或者接收数据。 channel本身必然是同时支持读写的，否则根本没法用。假如一个channel真的只能读，那么肯定只会是空的，因为你没机会往里面写数据。同理，如果一个channel只允许写，即使写进去了，也没有丝毫意义，因为没有机会读取里面的数据。所谓的单向channel概念，其实只是对channel的一种使用限制。

我们在将一个channel变量传递到一个函数时，可以通过将其指定为单向channel变量，从而限制 该函数中可 以对此 channel的操作， 比如只能往 这个 channel写，或者只 能从这个channel读。

单向channel变量的声明非常简单，如下：
```go
var ch1 chan int // ch1是一个正常的channel，不是单向的
var ch2 chan<- float64// ch2是单向channel，只用于写float64数据
var ch3 <-chan int // ch3是单向channel，只用于读取int数据
```
那么单向channel如何初始化呢？之前我们已经提到过， channel是一个原生类型，因此不仅支持被传递，还支持类型转换。只有在介绍了单向channel的概念后，读者才会明白类型转换对于channel的意义：就是在单向channel和双向channel之间进行转换。示例如下：
```go
ch4 := make(chan int)
ch5 := <-chan int(ch4) // ch5就是一个单向的读取channel
ch6 := chan<- int(ch4) // ch6 是一个单向的写入channel
```
基于ch4，我们通过类型转换初始化了两个单向channel：单向读的ch5和单向写的ch6。

为什么要做这样的限制呢？从设计的角度考虑，所有的代码应该都遵循“最小权限原则”，从而避免没必要地使用泛滥问题，进而导致程序失控。写过C++程序的读者肯定就会联想起const指针的用法。非const指针具备const指针的所有功能，将一个指针设定为const就是明确告诉函数实现者不要试图对该指针进行修改。单向channel也是起到这样的一种契约作用。

下面我们来看一下单向channel的用法：
```go
func Parse(ch <-chan int) {
    for value := range ch {
        fmt.Println("Parsing value", value)
    }
}
```
除非这个函数的实现者无耻地使用了类型转换，否则这个函数就不会因为各种原因而对ch进行写，避免在ch中出现非期望的数据，从而很好地实践最小权限原则。

7. 关闭channel
关闭channel非常简单，直接使用Go语言内置的close()函数即可：

`close(ch)`

在介绍了如何关闭channel之后，我们就多了一个问题：如何判断一个channel是否已经被关闭？我们可以在读取的时候使用多重返回值的方式：

`x, ok := <-ch`

这个用法与map中的按键获取value的过程比较类似，只需要看第二个bool返回值即可，如果返回值是false则表示ch已经被关闭。