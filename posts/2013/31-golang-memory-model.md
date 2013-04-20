---
date: 2013-04-20
layout: post
title: Go语言内容模型
description: Go memory model
categories:
- Blog
tags:
- Golang
- Concurrent Programming

---


{:toc}

# 名称定义
执行体 - Go里的Goroutine或Java中的Thread

# 背景介绍
内存模型的目的是为了定义清楚变量的读写在不同执行体里的可见性。理解内存模型在并发编程中非常重要，因为代码的执行顺序和书写的逻辑顺序并不会完全一致，甚至在编译期间编译器也有可能重排代码以最优化CPU执行, 另外还因为有CPU缓存的存在，内存的数据不一定会及时更新，这样对内存中的同一个变量读和写也不一定和期望一样。

和[Java的内存模型规范](http://www.infoq.com/cn/articles/java-memory-model-1?utm_source=infoq&utm_medium=related_content_link&utm_campaign=relatedContent_articles_clk)类似，Go语言也有一个内存模型，相对JMM来说，Go的内存模型比较简单，Go的并发模型是基于CSP（[Communicating Sequential Process](http://en.wikipedia.org/wiki/Communicating_sequential_processes)）的，不同的Goroutine通过一种叫Channel的数据结构来通信，Java的并发模型基于多线程和共享内存，有较多的概念（violatie, lock, final, construct, thread, atomic等）和场景，当然java.util.concurrent并发工具包大大简化了Java并发编程。

Go内存模型规范了一个Goroutine对某个变量的修改一定对其它Goroutine可见。


# Happens Before
在一个单独的Goroutine里，对变量的读写会和代码的顺序一致。比如以下的代码:

```
package main

import (
	"log"
)

var a, b, c int

func main() {
	a = 1
	b = 2
	c = a + 2
	log.Println(a, b, c)
}
```
尽管在编译期和执行期，编译器和CPU都有可能重排代码，比如，先执行b:=2，再执行a:=1，但c:=a+1是保证在a:=1后执行的。这样最后的执行结果一定是`1 2 3`，不会是`1 2 2`，但下面的代码则可能会输出`0 0 0`，`1 2 2`,  `0 2 3` (b=2比a=1先执行), `1 2 3`等各种可能。

```
package main

import (
	"log"
)

var a, b, c int

func main() {
	go func() {
		a = 1
		b = 2
	}()
	go func() {
		c = a + 2
	}()
	log.Println(a, b, c)
}
```

## Happens-before 定义
Happens-before用来指明Go程序里的内存操作的局部顺序。如果一个内存操作事件e1 happens-before e2，则e2 happens-after e1也成立；如果e1不是happens-before e2,也不是happens-after e2，则e1和e2是并发的。

在这个定义之下，如果以下情况满足，则对变量（v）的内存写操作（w）对一个内存读操作（r）来说**允许**可见的：

1. r在w不在w之前发生（可以是之后或并发）；
2. w和r之间没有另一个写操作(w')发生；

为了保证对变量（v）的一个特定写操作（w）对一个读操作（r）可见，就需要确保w是r**唯一****允许**的写操作，于是如果以下情况满足，则对变量（v）的内存写操作（w）对一个内存读操作（r）来说**保证**可见的：

1. w在r之前发生；
2. 所有其它对v的写操作只在w之前或r之后发生；

可以看出后一种约定情况比前一种更严格，这种情况要求没有w或r没有其他的并发写操作。

在单个Goroutine里，因为肯定没有并发，上面两种情况是等价的。对变量v的读操作可以读到最近一次写操作的值（这个应该很容易理解）。但在多个Goroutine里如果要访问一个共享变量，我们就必须使用同步工具来建立happens-before条件，来保证对该变量的读操作能读到期望的修改值。

**要保证并行执行体对共享变量的顺序访问方法就是用锁**。Java和Go在这点上是一致的。

以下是具体的可被利用的Go语言的happens-before规则，从本质上来讲，happens-before规则确定了CPU缓冲和主存的同步时间点（通过[内存屏障](http://hugozhu.myalert.info/2013/03/28/22-memory-barriers-or-fences.html)），从而使得对变量的读写可被确定完成。

# 同步方法

## 初始化

1. 如果package p 引用了package q，q的init()方法 happens-before p （Java工程师可以对比一下[final变量的happens-before规则](http://www.infoq.com/cn/articles/java-memory-model-6?utm_source=infoq&utm_medium=related_content_link&utm_campaign=relatedContent_articles_clk)）
2. main.main()方法 happens-after所有package的init()方法结束。

## Goroutine创建

1. **go语句创建新的goroutine happens-before 该goroutine执行**（这个应该很容易理解）

```
package main

import (
	"log"
	"time"
)

var a, b, c int

func main() {
	a = 1
	b = 2
	go func() {
		c = a + 2
		log.Println(a, b, c)
	}()
	time.Sleep(1 * time.Second)
}
```
利用这条happens-before，我们可以确定`c=a+2`是happens-after`a=1和b=2`，所以结果输出是可以确定的`1 2 3`，但如果是下面这样的代码，输出就不确定了，有可能是`1 2 3`或`0 0 2`

```
func main() {
	go func() {
		c = a + 2
		log.Println(a, b, c)
	}()
	a = 1
	b = 2
	time.Sleep(1 * time.Second)
}
```

## Goroutine销毁

1. **Goroutine的退出并不保证happens-before任何事件**。

```
var a string

func hello() {
	go func() { a = "hello" }()
	print(a)
}
```
上面代码因为`a="hello"` 没有使用同步事件，并不能保证这个赋值被主goroutine可见。事实上，极度优化的Go编译器甚至可以完全删除这行代码`go func() { a = "hello" }()`。

Goroutine对变量的修改需要让对其它Goroutine可见，除了使用锁来同步外还可以用Channel。

## Channel通信
在Go编程中，Channel是被推荐的执行体间通信的方法，Go的编译器和运行态都会尽力对其优化。

1. **对一个Channel的发送事件(send) happens-before 相应Channel的接收事件完成**
2. **关闭一个Channel happens-before 从该Channel接收返回0**
3. **不带缓冲的Channel的接收 happens-before 相应Channel的发送事件完成**

```
var c = make(chan int, 10)
var a string

func f() {
	a = "hello, world"
	c <- 0
}

func main() {
	go f()
	<-c
	print(a)
}
```
上述代码可以确保输出`hello, world`，因为`a = "hello, world"` happens-before `c <- 0`，`print(a)` happens-after `<-c`， 根据上面的规则1）以及happens-before的可传递性，`a = "hello, world"` happens-before`print(a)`。

```
var c = make(chan int, 1)
var a string

func f() {
	a = "hello, world"
	<-c
}

func main() {
	go f()
	c <- 0
	print(a)
}
```
根据规则3），因为c是不带缓冲的Channel，`a = "hello, world"` happens-before `<-c` happens-before `c <- 0` happens-before `print(a)`， 但如果c是缓冲队列，那结果就不确定了。

# 锁
`sync` 包实现了两种锁数据结构:

1. sync.Mutex -> java.util.concurrent.ReentrantLock
2. sync.RWMutex -> java.util.concurrent.locks.ReadWriteLock

其happens-before规则和Java的也类似：

1. **任何sync.Mutex或sync.RWMutex 变量（l），定义 n < m， 第n次 `l.Unlock()` happens-before 第m次`l.lock()`调用返回。**

```
var l sync.Mutex
var a string

func f() {
	a = "hello, world"
	l.Unlock()
}

func main() {
	l.Lock()
	go f()
	l.Lock()
	print(a)
}
```
`a = "hello, world"` happens-before `l.Unlock()` happens-before 第二个 `l.Lock()` happens-before `print(a)`

# Once
`sync`包还提供了一个安全的初始化工具Once。还记得Java的Singleton设计模式，double-check，甚至triple-check的各种单例初始化方法吗？Go则提供了一个标准的方法。

1. `once.Do(f)`中的`f()` happens-before 任何多个once.Do(f)调用的返回，且f()有且只有一次调用。

```
var a string
var once sync.Once

func setup() {
	a = "hello, world"
}

func doprint() {
	once.Do(setup)
	print(a)
}

func twoprint() {
	go doprint()
	go doprint()
}
```

上面的代码虽然调用两次`doprint()`，但实际上setup只会执行一次，并且并发的`once.Do(setup)`都会等待setup返回后再继续执行。


# 参考链接

1. http://golang.org/ref/mem
2. http://en.wikipedia.org/wiki/Java_Memory_Model
3. http://ifeve.com/java-memory-model-1/
4. http://code.google.com/p/golang-china/wiki/go_mem#Channel_communication_管道通信