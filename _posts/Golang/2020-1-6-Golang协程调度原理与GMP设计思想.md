---
title: Golang协程调度原理与GMP设计思想
category: Golang
typora-root-url: ../..
---

# 调度原理



## 调度器的由来

![image-20210106194226598](/assets/img/image-20210106194226598.png)



减少CPU调度的消耗和内存占用 ，缺点是协程调度器实现复杂

![image-20210106194528553](/assets/img/image-20210106194528553.png)