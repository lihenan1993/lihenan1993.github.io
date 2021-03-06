---
title: Go语言并发之道
category: 读书笔记
typora-root-url: ../..
---

# 第一章 并发概述

### 现代编程遇到的难题

云和分布式的普及，使得并发编程建模很难

### 难点

- 竞争条件

  当两个或者多个操作必须按正确的顺序执行，而程序并未保证这个顺序，就会发生竞争条件

  ```go
  var data int
  go func() {
      data++
  }
  if data==0 {
      fmt.Printf("the value is %v.\n", data)
  }
  ```

  该程序执行有三个结果：

  1. 不打印
  2. 0
  3. 1

- 引入数据竞争的原因

  - 开发人员以顺序执行的思维思考问题

  > 现在回看，这个太片面了。
  >
  > 应该是开发人员没有真正理解程序底层是如何执行的，
  >
  > 表面是多个goroutine同时访问临界资源，
  >
  > 底层来说由于CPU缓冲，内存重排，编译器重排，导致程序执行顺序与代码顺序不一致。

- 对策

  - 仔细遍历所有可能的方案
  - 思考时候，**拉长相对时间**
  - 休眠能使竞争条件发生概率降低，但是不能消除，应该以逻辑正确性为目标

  > golang里面，可以加锁，或者用chan进行通讯

### 原子性

运行不可分割、不可中断、

考虑原子性：

- 一定要在一个上下文中考虑原子性。
  - 因为一个原子操作可能在应用程序中是原子性的，但是在操作系统中不是。

### 临界区

多个goroutine同时访问的一块内存区域

加锁

- 会使得程序运行变慢

## 死锁、活锁和饥饿

### 死锁

所有并发进程彼此等待，没有外界干预的情况下，程序永远无法恢复

大多数时候，golang能检测输出死锁（fatel error: all goroutines are asleep - deadlock!）

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

type value struct {
	mu sync.Mutex
	v int
}

var wg sync.WaitGroup

func printValue(a,b *value, wg * sync.WaitGroup)  {
	defer wg.Done()
	a.mu.Lock()
	defer a.mu.Unlock()

	time.Sleep(2 * time.Second)

	b.mu.Lock()
	defer b.mu.Unlock()

	fmt.Println(a.v,b.v)
}

func main()  {
	var a, b value
	wg.Add(2)

	go printValue(&a,&b,&wg)
	go printValue(&b,&a,&wg)
	wg.Wait()
}

```

死锁原因，第一个goroutine锁a准备锁b，但是第二个goroutine锁b准备锁a，wg使得两个goroutine执行完程序才能结束，这种情况两个进程没法继续执行。

死锁发生条件：

1. 相互排斥：并发进程同时拥有资源的独占权
2. 等待条件：并发进程必须同时拥有一个资源，并等待额外的资源
3. 没有抢占：并发进程拥有的资源只能被该进程释放
4. 循环等待：p1等待p2,p2也等待p1

### 活锁

活锁是正在主动执行并发操作的程序，但这些操作无法向前推进。

```go
package main

import (
	"bytes"
	"fmt"
	"sync"
	"sync/atomic"
	"time"
)

func main() {
	takeStep := func() {
		time.Sleep(time.Second)
	}

	tryDir := func(dirName string, dir *int32, out *bytes.Buffer) bool {
		fmt.Fprintf(out, "%+v", dirName)
		// 方向人数+1
		atomic.AddInt32(dir, 1)
		// 等待一秒钟
		takeStep()
		// 走成功就返回
		if atomic.LoadInt32(dir) == 1 {
			fmt.Fprint(out, ".Success!")
			return true
		}
		// 等待一秒钟
		takeStep()
		//没走成功，再走回来
		atomic.AddInt32(dir, -1)

		return false
	}

	var left, right int32

	tryLeft := func(out *bytes.Buffer) bool {
		return tryDir("向左走 ", &left, out)
	}

	tryRight := func(out *bytes.Buffer) bool {
		return tryDir("向右走 ", &right, out)
	}

	walk := func(walking *sync.WaitGroup, name string) {
		var out bytes.Buffer
		defer walking.Done()
		defer func() { fmt.Println(out.String()) }()
		fmt.Fprintf(&out, "%v is tring to scoot:", name)
		for i := 0; i < 5; i++ {
			if tryLeft(&out) || tryRight(&out) {
				return
			}
		}

		fmt.Fprintf(&out, "\n%v is tried !", name)
	}

	var trail sync.WaitGroup
	trail.Add(2)
	go walk(&trail, "男人")
	go walk(&trail, "女人")
	trail.Wait()
}
```



```
男人 is tring to scoot:向左走 向右走 向左走 向右走 向左走 向右走 向左走 向右走 向左走 向右走 
男人 is tried !
女人 is tring to scoot:向左走 向右走 向左走 向右走 向左走 向右走 向左走 向右走 向左走 向右走 
女人 is tried !
```

### 饥饿

饥饿：表示在任何情况下，并发进程都无法获得执行工作所需的所有资源
饥饿通常指一个或多个并发进程占有资源，使得其他进程不能占有资源进行执行

饥饿会导致程序性能不佳或错误，有可能使得程序出错，如果上面的例子中greedy完全阻止了polite完成工作，会使得polite永远得不到执行
在进行内存访问同步时，需要在粗粒度同步和细粒度同步之间找到平衡点，以提高程序的性能

```go
var wg sync.WaitGroup
var sharedLock sync.Mutex
const runtime = 1 * time.Second
greedyWorker := func(){
    defer wg.Done()
    var count int
    for begin := time.Now(); time.Since(begin) <= runtime; {
        sharedLock.Lock()
        time.Sleep(3*time.Nanosecond)
        sharedLock.Unlock()
        count++
    }
    fmt.Printf("Greedy worker was able to execute %d work loops\n",count)
}
politeWorker := func(){
    defer wg.Done()
    var count int
    for begin := time.Now(); time.Since(begin) <= runtime; {
        sharedLock.Lock()
        time.Sleep(1*time.Nanosecond)
        sharedLock.Unlock()
        sharedLock.Lock()
        time.Sleep(1*time.Nanosecond)
        sharedLock.Unlock()
        sharedLock.Lock()
        time.Sleep(1*time.Nanosecond)
        sharedLock.Unlock()
        count++
    }
    fmt.Printf("Polite worker was able to execute %d work loops\n",count)
}
wg.Add(2)
go greedyWorker()
go politeWorker()
wg.Wait()

结果： 
Greedy worker was able to execute 297 work loops
Polite worker was able to execute 99 work loops
从结果中可以看出greedyWorker扩大了其持有共享锁的临界区，并阻止了politeWorker的高效工作
```







# 第二章 通信顺序进程 CSP

### 并发和并行概念

并发：同一时间段内执行多个操作。代码层面

并行：同一时刻执行多个操作。代码，操作系统，CPU等相关



### 什么是CSP

CSP 即 “Communicating Sqquential processes” (通信顺序进程) 1978年论文

为golang提供并发原语

golang减少并发问题建模的复杂性，复杂性的减少带来的是更少的bug，更高的开发效率



### Go语言的并发哲学

CSP原语和内存访问同步（sync包）都可以选择，使用最好描述和最简单的那个方式

但是推荐用CSP

座右铭：”通过通信共享内存，而不是通过共享内存来通信“



### channel OR mutex(互斥)

![channel OR mutex](/assets/img/20200920173939838.png)

1. 这是一个性能要求很高的临界区吗？

   channel会稍微慢点，但绝不是要因为更高性能，必须选mutex，可能重新规划程序更好。

2. 想要转让数据的所有权吗？

   例如将运算结算给另一个代码块

   生产者消费者

3. 保护某个结构的内部状态

   结构体中某个变量的增加。类似原子操作。

4. 协调多个逻辑片段

   channel肯定比sync更具有组合型



- 总结
  1. 关注数据的流动，就可以使用channel解决并发问题。
  2. 不流动的数据，如果存在并发访问，尝试使用sync.Mutex保护数据。
  3. 对于大问题，channel + mutex也许才是更好的方案。
