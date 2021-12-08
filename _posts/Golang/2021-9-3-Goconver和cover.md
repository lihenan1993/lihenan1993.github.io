---
title: Goconver和cover
category: Golang
typora-root-url: ../../
---

 

## Goconver

1. 介绍

   测试框架

   兼容go test

   全自动web ui

   输出更有号

   显示覆盖率

   代码简洁

2. 安装

   ```
   go get github.com/smartystreets/goconvey
   ```

3. 调用

```
package theme

import "testing"
import . "github.com/smartystreets/goconvey/convey"

func TestAdd(t *testing.T) {
	Convey("两数相加", t, func() {
		So(Add(1, 2), ShouldEqual, 3)
	})
}
```

4. 开启web结果

   goconvey

   http://127.0.0.1:8080

## cover

1.

```
go test -v -covermode=set -coverprofile=coverage.out -coverpkg ../...
go tool cover -html=coverage.out
```

2.

```
go test -v -covermode=count -coverprofile=coverage.out -coverpkg ./...

```

go test指令中新增了covermode, coverprofile, coverpkg 三个参数

- covermode可以设置3个值

```
set: 只包含某一行是否被执行。
count: 某一行被执行过多少次
atomic: 同count，但是用于并发的场景
```

一般就是设置成count，可以统计代码行被执行了几次。

- coverprofile就是设置覆盖信息的输出文件，覆盖信息包含了哪些行被执行以及执行了几次的信息。

- coverpkg是列举出要统计覆盖率的包，./…代表当前目录下的所有包，含递归的。





## 静态检测

```
golangci-lint run -c .golangci.yml ./theme/beerfest/theme_beerfest.go
```

