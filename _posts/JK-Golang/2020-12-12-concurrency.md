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

1. CPU内存重排和编译器重排

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



## Package sync



## chan



## Package context



