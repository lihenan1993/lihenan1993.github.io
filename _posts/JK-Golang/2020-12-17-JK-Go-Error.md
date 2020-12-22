---
title: 第02周：Error
category: JK-Go
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
   - 生产环境中，资源依赖初始化，如果是弱依赖的最好别panic，强依赖的可以panic
   
   - 运行中，野生goroutine的一定要recover，最好是基础库封装go
   
     ```go
     func Go(f func()) {
         go func() {
             defer func() {
                 if err := recover(); err != nil {
                     fmt.Println(err)
                 }
             }()
             
             f()
         }()
     }
     ```
   
     核心
   
     ```go
     defer func() {
         if err := recover(); err != nil {
             // handle error
         }
     }()
     ```
   
     
   
   - 千万不能随意panic



### go中错误处理的特点

1. 简单
2. 考虑失败，而不是成功
3. 没有隐藏的控制流
4. 完全交给你来控制error
5. Error are values



## Error Type

1. 预定义的特定错误

   io.EOF

   不灵活

   会成为公共API的一部分

   在两个包之间创建了依赖

2. 实现了Error接口的自定义类型

   ```go
   type MyError struct {
       Msg  string
       File string
       Line int
   }
   
   func (e *MyError) Error() string {
       return fmt.Sprintf("%s:%d: %s",e.File, e.Line, e.Msg)
   }
   ```

   调用者通过断言判断错误类型，获取更多上下文

   ![image-20201219175655201](/assets/img/image-20201219175655201.png)

   调用者要使用类型断言和类型 *switch*，就要让自定义的 *error* 变为 公共的。

   这种模型会导致和调用者产生强耦合，从而导致 API 变得脆弱

   3. 只返回错误，不假设其内容

      ```go
      func fn() error {
          x, err := bar.Foo()
          if err != nil {
              return err
          }
          // do something
      }
      ```

      

# Handling Error

1. 处理错误时候，异常逻辑直接抛出，正常逻辑继续写

```go
f, err := os.Open(path)
if err != nil {
    // handle error
}
// do stuff
```

2. 利用for迭代和库的支持

   ```go
   func CountLines(r io.Reader) (int,error)  {
   	sc := bufio.NewScanner(r)
   	lines := 0
   	for sc.Scan() {
   		lines++
   	}
   
   	return lines,sc.Err()
   }
   ```

3. 用一个内部类型 加 err error的结构体 返回错误

```go
type errWriter struct {
	io.Writer
	err error
}

func (e *errWriter)Write(buf []byte)(int,error)  {
	if e.err != nil {
		return 0,e.err
	}

	return e.Writer.Write(buf)
}
```

# Wrap error

1. 你应该只处理错误一次。处理错误意味着检查错误的值，并做出简单的选择（打日志、mock数据）。

2. 出错之后，如果要吞掉error，请对value负责，进行服务降级，或者返回mock数据

3. 日志记录与错误无关且对调试没有帮助的信息应被视为噪音，应予以质疑。记录的原因是因为某些东西失败了，而日志包含了答案。
4. 错误不重复报告

```go
package main

import (
"fmt"
"github.com/pkg/errors"
)

func main()  {
	err := errors.Wrapf(errors.New("1.最初的错误原因"),"%s","2.包裹一层")
	err = errors.WithMessage(err,"3.这里也包裹一层，但是不添加调用栈的信息，只追加错误信息")
	fmt.Printf("错误类型：%T\n错误信息：%v\n调用栈：\n%+v\n",errors.Unwrap(err),errors.Unwrap(err),err)
}
```

```
错误类型：*errors.withStack
错误信息：2.包裹一层: 1.最初的错误原因
调用栈：
1.最初的错误原因
main.main
	D:/work/learn/code/error/main.go:9
runtime.main
	D:/Go/src/runtime/proc.go:204
runtime.goexit
	D:/Go/src/runtime/asm_amd64.s:1374
2.包裹一层
main.main
	D:/work/learn/code/error/main.go:9
runtime.main
	D:/Go/src/runtime/proc.go:204
runtime.goexit
	D:/Go/src/runtime/asm_amd64.s:1374
3.这里也包裹一层，但是不添加调用栈的信息，只追加错误信息

Process finished with exit code 0

```

#### Warp使用准则

1. 只有自己的应用程序代码，最初发生错误的时候要warp error
2. 如果是做的是基础库，只简单返回预定义错误
3. 直接返回错误，别到处打日志
4. 在程序的顶部或者是工作的goroutine顶部（请求入口），使用%+v把堆栈详情记录
5. 使用 errors.Cause 获取 root error ，再和预定义错误进行判断
6. 如果不打算处理错误，就warp回去，上下文中添加输入参数或者失败的执行语句
7. 如果错误处理过，打过日志，或者做了降级处理，那么不该再往上抛 



## Go1.13 Error

最大的问题是不包含调用栈

```go
package main

import (
	"database/sql"
	"errors"
	"fmt"
	"os"
)

func main()  {
	err := fmt.Errorf("%w",errors.New("最初的错误原因"))
	err = fmt.Errorf("包裹一层:%w",err)
	
	// 不包含调用栈
	// outPut:  包裹一层:最初的错误原因
	fmt.Printf("%+v",err)
	
	// Is是判断错误链里面是否包含指定错误
	if errors.Is(err,sql.ErrNoRows) {
		// pass
	}

	// As是帮你分析错误链，找到指定的错误，类似java里面的catch(e RuntimeException)
	if _, err := os.Open("non-existing"); err != nil {
		var pathError *os.PathError
		if errors.As(err, &pathError) {
			fmt.Println("Failed at path:", pathError.Path)
		} else {
			fmt.Println(err)
		}
	}
}
```

