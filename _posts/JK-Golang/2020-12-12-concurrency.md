---
title: concurrency
category: JK-Go
typora-root-url: ../..
---

 ## Goroutine

进程：操作系统给每个应用一个进程

主线程：main函数

普通线程：操作系统调度不同的线程

使用原则：

1. 让你自己忙或自己做这项工作
   1. 如果你的 goroutine 在从另一个 goroutine 获得结果之前无法取得进展，那么通常情况下，你自己去做这项工作比委托它( go func() )更简单。
2. 不要启动一个goroutine却不知道他什么时候结束
   1. 确定什么时候结束
   2. 如何结束
3. log.Fatal 结束程序的时候，不会执行defer ，所以只能在main.main 或者 init 函数中使用
4. 并发的逻辑给调用者，让调用者决定是否要并发。不要在函数内部开启goroutine。
   1. 使用标准库  filepath.WalkDir 的做法

总结：

1. 让调用者决定是否开启go

2. 开启后要管控goroutine的生命周期

3. 函数要能够控制什么时候退出，用chan 或者 context都行

   

## Memory model

1. 两个goroutine同时读或者修改一个变量，先后执行顺序无法预测

原因：

1. CPU缓存、内存重排和编译器重排

   现代 CPU 为了“抚平” 内核、内存、硬盘之间的速度差异，搞出了各种策略，例如三级缓存等。为了让 (2) 不必等待 (1) 的执行“效果”可见之后才能执行，我们可以把 (1) 的效果保存到 store buffer：

   ```go
   // Thread 1
   go func() {
       A = 1
       fmt.Print(B)
   }
   
   // Thread 2
   go func() {
       B = 1
       fmt.Print(A)
   }
   
   //结果可能是
   1 0
   0 1
   1 1
   // 还可能是
   0 0
   ```

   解释 产生0 0 的原因：

   更改只刷到了CPU的缓存，没有写入内存，所以内存中其他线程读还是0

   ![image-20201223204645391](/assets/img/image-20201223204645391.png)

https://www.cnblogs.com/egmkang/p/14080645.html

建议：

1. 如果程序中修改数据时有其他goroutine同时读取，那么必须将读取串行化。
2. 为了串行化访问，请使用channel或其他同步原语，例如sync和sync/atomic来保护数据。

#### 先行发生

为了说明读和写的必要条件，我们定义了`先行发生（Happens Before）`--Go程序中执行内存操作的偏序。如果事件`e1`发生在`e2`前，我们可以说`e2`发生在`e1`后。如果`e1`不发生在`e2`前也不发生在`e2`后，我们就说`e1`和`e2`是并发的。

1. w先行发生于r
2. 其他对共享变量v的写操作要么在w前，要么在r后。



#### channel先行发生原则 

1. 无缓冲channel的读 先行发生于 写

   ```go
   var c = make(chan int)       // 无缓冲
   var a string
   
   func f() {
       a = "hello, world"
       <-c                  // 读，先
   }
   
   func main() {
       go f()
       c <- 0              // 写，后
       print(a)
   }
   ```

2. 有缓冲channel的写 先行发生于 读

   ```go
   var c = make(chan int,1)  // 有缓冲
   var a string
   
   func f() {
   	a = "hello, world"
   	c <- 0				// 写，先
   }
   
   func main() {
   	go f()
   	<-c                 // 读，后
   	print(a)
   }
   ```

3. 总结

   无缓冲 (关闭 > 读 > 写)

   ​	读 > 写

   ​	关闭 > 读

   ​	关闭 > 写 （panic）  

   有缓冲 （关闭，写 > 读）

   ​	写 > 读

   ​	关闭 > 读

   ​	写 <> 关闭 （缓冲用完 panic）

   记忆方法：

   1. 有缓冲和无缓冲 读写相反，有缓冲先写 ，无缓冲先读



#### 如何优雅地关闭channel

1. 多次close(ch) ，会panic
2. 向关闭的通道写入，会panic
3. 没有通用而简单的方法检查通道是否关闭而不修改通道的状态

通道关闭原则

不要在数据接收方或者在有多个发送者的情况下关闭通道

我们只应该让一个通道唯一的发送者关闭此通道。

有接收者遍历通道的时候，记得关闭通道



解决办法：

 	1. 一个发送者，多个接收者，则由发送者来关闭通道
 	2. 否则，引入主持人（stop channel）来关闭通道

> https://go101.org/article/channel-closing.html
>
> https://www.cnblogs.com/sunlong88/p/13590845.html

3. recover发生的panic

   ```go
   type MyChannel struct {
       C    chan T
       once sync.Once
   }
   
   func NewMyChannel() *MyChannel {
       return &MyChannel{C: make(chan T)}
   }
   
   func (mc *MyChannel) SafeClose() {
       mc.once.Do(func() {
           close(mc.C)
       })
   }
   ```

   

## Package sync

1. 不要通过共享内存来通信，而要通过通信来共享内存
2. 数据竞争 data race : 两个及以上goroutine对同一资源的读写
   1. 用 `go build -race` 或 `go test -race`检测
   2. 因为赋值非原子操作，可能产生上下文切换

3. 很多操作看似是原子的，其实汇编不是原子操作

   ```go
   package main
   
   func main() {
   	var i float64
   	for i<1000 {
   		i++
   	}
   }
   ```

   查看`go tool compile -S plus.go`可以发现，赋值是两步操作，这时候上下文切换，会导致读到错误的值

   ```
   0x0005 00005 (plus.go:6)        MOVSD   $f64.3ff0000000000000(SB), X2
   0x000d 00013 (plus.go:6)        ADDSD   X2, X0
   ```



### atomic.Value

**读多写少的场合使用**

> https://studygolang.com/articles/23242?fr=sidebar

Go 语言在 1.4 版本的时候向`sync/atomic`包中添加了一个新的类型`Value`。此类型的值相当于一个容器，可以被用来“原子地"存储（Store）和加载（Load）**任意类型**的值。

1. 更快

   `Mutex`由**操作系统**实现，而`atomic`包中的原子操作则由**底层硬件**直接提供支持。在 CPU 实现的指令集里，有一些指令被封装进了`atomic`包，这些指令在执行的过程中是不允许中断（interrupt）的，因此原子操作可以在`lock-free`的情况下保证并发安全，并且它的性能也能做到随 CPU 个数的增多而线性扩展。

#### 使用方法

`atomic.Value`类型对外暴露的方法就两个：

- `v.Store(c)` - 写操作，将原始的变量`c`存放到一个`atomic.Value`类型的`v`里。
- `c = v.Load()` - 读操作，从线程安全的`v`中读取上一步存放的内容。

```go
package main

import (
	"fmt"
	"sync/atomic"
	"time"
)

var i = 0

func loadConfig() int {
	// 从数据库或者文件系统中读取配置信息
	i = i+1
	return i
}

func main() {
	// config变量用来存放该服务的配置信息
	var config atomic.Value
	// 初始化时从别的地方加载配置文件，并存到config变量里
	config.Store(loadConfig())
	go func() {
		// 每10秒钟定时的拉取最新的配置信息，并且更新到config变量里
		for {
			time.Sleep(20 * time.Millisecond)
			// 对应于赋值操作 config = loadConfig()
			config.Store(loadConfig())
		}
	}()
	// 创建工作线程，每个工作线程都会根据它所读取到的最新的配置信息来处理请求
	for i := 0; i < 10; i++ {
		go func() {
			for {
				time.Sleep(time.Second)
				// 对应于取值操作 c := config
				// 由于Load()返回的是一个interface{}类型，所以我们要先强制转换一下
				c := config.Load().(int)
				// 这里是根据配置信息处理请求的逻辑...
				fmt.Println(c)
			}
		}()
	}

	time.Sleep(time.Minute)
}
```

### Mutex机制

```go
func main() {
	done := make(chan bool,1)
	var mu sync.Mutex

	// goroutine 1
	go func() {
		for {
			select {
			case <-done:
				return
			default:
				mu.Lock()
				time.Sleep(100 * time.Microsecond)
				mu.Unlock()
			}
		}
	}()

	// goroutine 2
	for i:=0; i<10; i++ {
		time.Sleep(100 * time.Microsecond)
		mu.Lock()
		mu.Unlock()
	}

	done <- true
}
```

go 1.8

![image-20201224174224146](/assets/img/image-20201224174224146.png)

go 1.9

![image-20201224174236583](/assets/img/image-20201224174236583.png)

#### mutex模式

- Barging. 这种模式是为了提高吞吐量，当锁被释放时，它会唤醒第一个等待者，然后把锁给第一个等待者或者给第一个请求锁的人。

- Handsoff. 当锁释放时候，锁会一直持有直到第一个等待者准备好获取锁。它降低了吞吐量，因为锁被持有，即使另一个 goroutine 准备获取它。

  *一个互斥**锁的* *handsoff* *会完美地平衡两个goroutine* *之间的锁分配，但是会降低性能，因为它会迫使**第一个* *goroutine* *等待锁**。*

- Spinning. 自旋在等待队列为空或者应用程序重度使用锁时效果不错。Parking 和 Unparking goroutines 有不低的性能成本开销，相比自旋来说要慢得多



#### 结果原因

Go 1.8 使用了 Barging 和 Spining 的结合实现。当试图获取已经被持有的锁时，如果本地队列为空并且 P 的数量大于1，goroutine 将自旋几次(用一个 P 旋转会阻塞程序)。自旋后，goroutine park。在程序高频使用锁的情况下，它充当了一个快速路径。

Go 1.9 通过添加一个新的饥饿模式来解决先前解释的问题，该模式将会在释放时候触发 handsoff。所有等待锁超过一毫秒的 goroutine(也称为有界等待)将被诊断为饥饿。当被标记为饥饿状态时，unlock 方法会 handsoff 把锁直接扔给第一个等待者。

在饥饿模式下，自旋也被停用，因为传入的goroutines 将没有机会获取为下一个等待者保留的锁。



总结：

有饥饿的，先给饥饿的goroutine锁，更公平，吞吐量低了。



### errGroup

*https://github.com/go-kratos/kratos/tree/master/pkg/sync/errgroup*

并发执行多个function ，并把结果返回。

一个出错，可以取消所有子goroutine，也可以不取消

捕获野生的panic

控制并发量



### sync.Pool

**sync.Pool** **的场景是用来保存和复用临时对象，以减少内存分配，降低** **GC** **压力**

不能用于连接池，因为池内对象会被随时回收。

用于经常需要开辟空间，并用完就可抛弃的变量。





## chan

channel 是一种类型安全的消息队列，充当两个goroutine之间的管道，通过它进行资源交换。

分类：无缓冲和有缓冲

1. 无缓冲

   无缓冲chan没有容量，因此交换前两个goroutine要准备好

   发送没有接受，或者接受没有发送，会锁住通道

   特点：

   接受先于发送

   好处： 100%保证能收到

   代价：延迟时间未知

2. 有缓冲

   有容量

   满的时候阻塞接收，空的时候阻塞发送

   缓冲区大小可能极大的影响性能。

   例如 fan-out 模式中，一个写入，多个读取，缓冲区越大，越会产生goroutine的上下文切换，浪费性能

   ![image-20201227115257177](/assets/img/image-20201227115257177.png)

   特点：

   发送先于接收

   好处： 延迟更小

   代价：不保证数据到达



## Package context

总结：

1. 返回的cancel方法，一定要记得defer cancel()，否则会内存泄漏

2. 服务端的传入请求要创建context

3. 服务端传出的调用要接受context

4. 不要在struct中传递context，应该作为首参数

5. 使用 WithCancel, WithDeadline, WithTimeout, or WithValue 来替换context

6. 当一个context被取消，其子context也应该被取消

7. context可以传递给多个goroutine使用

8. 不要传递nil的context，可以传TODO

9. 仅对API的请求范围数据使用上下文，而不是可选参数

   

解决的问题：

go服务需要根据 请求中特定的值，去调用其他请求（数据库，rpc等），当一个请求超时或者过期，应该取消其他请求。

#### 传递方式

1. 首参数传递

   ```go
   func rpc1(ctx context.Context, 其他参数) {
       select {
   		case <-ctx.Done():
   			fmt.Println("清空资源")
   			return
   		default:
   			fmt.Println("default")
   			time.Sleep(time.Second)
       }
   ```

2. 通过一个给定的context 返回新的对象。例如：net/http库的Request.WitchContext

   ```
   func (r *Request) WithCOntext(ctx context.Context) *Request
   ```

注意：

**使用** **context** **的一个很好的心智模型是它应该在程序中流动，应该贯穿**你的代码。这通常意味着您不希望将其存储在结构体之中。它从一个函数传递到另一个函数，并根据需要进行扩展。理想情况下，每个请求都会创建一个** **context** **对象，并在请求结束时过期。**



#### WitchValue

1. 用WithValue来传递值和更新值

```go
package main

import (
	"context"
	"fmt"
)

type test01String string
type test02String string

func main() {
	// context实例
	ctx := context.Background()
	test01(ctx)
}

func test01(ctx context.Context) {
	// context实例
	ctx01 := context.WithValue(ctx, "key1", "hello1")
	test02(ctx01)
}

func test02(ctx context.Context) {
	// context实例
	ctx02 := context.WithValue(ctx, "key1", "hello2")
	test03(ctx02)
}

func test03(ctx context.Context) {
	fmt.Println(ctx.Value("key1"))
}
```

2. value源码

简单的递归解析

```go
func WithValue(parent Context, key, val interface{}) Context {
	if parent == nil {
		panic("cannot create context from nil parent")
	}
	if key == nil {
		panic("nil key")
	}
	if !reflectlite.TypeOf(key).Comparable() {
		panic("key is not comparable")
	}
	return &valueCtx{parent, key, val}
}

// A valueCtx carries a key-value pair. It implements Value for that key and
// delegates all other calls to the embedded Context.
type valueCtx struct {
	Context
	key, val interface{}
}

func (c *valueCtx) Value(key interface{}) interface{} {
	if c.key == key {
		return c.val
	}
	return c.Context.Value(key)
}
```



3. 传递的值应该是元数据，而且不能所以修改，要修改用新的对象包裹旧的对象。copy on write

同一个 context 对象可以传递给在不同 goroutine 中运行的函数；

上下文对于多个 goroutine 同时使用是安全的。

对于值类型最容易犯错的地方，在于 context value 应该是 immutable 的，每次重新赋值应该是新的 context，即: 

context.WithValue(ctx, oldvalue)

4. **Replace a Context using WithCancel, WithDeadline, WithTimeout, or WithValue**

![image-20201226202409662](/assets/img/image-20201226202409662.png)

5. 用context value中携带timeout ,去更新子goroutine的生命周期，防止内存泄漏，goroutine泄漏