---
title: idea_debug_
category: Golang
typora-root-url: ../../
---

 Version of Delve is too old for this version of Go (maximum supported version 1.13, suppress this error





升级Golang版本后发现

```txt
Version of Delve is too old for this version of Go (maximum supported version 1.12, suppress this error with --check-go-version=false)
```



debug要升级一下

```
升级dlv
go get -u github.com/go-delve/delve/cmd/dlv

$gopath/bin/dlv
复制到

D:\Program Files\JetBrains\GoLand 2019.2.2\plugins\go\lib\dlv\windows
```

