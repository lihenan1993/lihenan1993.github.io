---
title: 第02周：Error
category: JK-Go进阶训练营
typora-root-url: ..
---

# Error

在go中，Error就是一个普通的接口定义

```go
// D:/Go/src/builtin/builtin.go

// The error built-in interface type is the conventional interface for
// representing an error condition, with the nil value representing no error.
type error interface {
	Error() string
}
```

通常用errors.New() 来创建一个error对象

```go
D:/Go/src/errors/errors.go

// New returns an error that formats as the given text.
// Each call to New returns a distinct error value even if the text is identical.
func New(text string) error {
	return &errorString{text}
}

// errorString is a trivial implementation of error.
type errorString struct {
	s string
}

func (e *errorString) Error() string {
	return e.s
}
```

> 注意，这里返回的是内部errorString的指针
>
> 如果返回字符串，判断等于的时候，可能会因为string相同，发生错误的判断

![image-20201217195410896](/assets/img/image-20201217195410896.png)

返回值：

Named Type Error



## Error vs Exception

- C

  单返回值，或者传递指针参数，返回执行成功还是失败

- C++

  有exception但是不知道调用方会抛出什么异常

- Java

  有错误类型，但是有人会把普通错误，抛出为RuntimeException的灾难级别



GO处理异常的逻辑

1. 返回value,error，当有error返回时候，一定要处理error
2. Panic就是挂掉了，不能让调用者处理
   - 生产环境中，不能panic，一定要recover
   - 

