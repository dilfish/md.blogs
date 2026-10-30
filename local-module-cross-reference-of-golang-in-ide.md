---
title: "Golang 本地包互相依赖在 IDE 中的显示问题"
date: 2022-01-11T20:51:18+08:00
draft: false
---

### 基本情况

例如我们的项目依赖三个本地的库，分别是 inta, intb, intc，其中，inta, intb 又依赖 intc。

```
➜   gomodcrossref 😈 tree
.
├── go.mod
├── gomodcrossref
├── internal
│   └── github.com
│       └── dilfish
│           ├── inta
│           │   ├── go.mod
│           │   └── main.go
│           ├── intb
│           │   ├── go.mod
│           │   └── main.go
│           └── intc
│               ├──  main.go
│               └── go.mod
└── main.go

6 directories, 9 files
```

其中项目代码和 go.mod 文件为：
```
➜   gomodcrossref 😈 cat main.go
package main

import (
	"github.com/dilfish/inta"
	"github.com/dilfish/intb"
)

func main() {
	inta.P()
	intb.P()
}
```
```
➜   gomodcrossref 😈 cat go.mod
module github.com/dilfish/gomodcrossref

replace github.com/dilfish/inta => ./internal/github.com/dilfish/inta

replace github.com/dilfish/intb => ./internal/github.com/dilfish/intb

replace github.com/dilfish/intc => ./internal/github.com/dilfish/intc

go 1.17

require (
	github.com/dilfish/inta v0.0.0-00010101000000-000000000000
	github.com/dilfish/intb v0.0.0-00010101000000-000000000000
)

require github.com/dilfish/intc v0.0.0-00010101000000-000000000000 // indirect
```

可以看到，直接依赖 inta, intb，对于 intc 的依赖是间接的。
inta 的代码如下：
```
➜   gomodcrossref 😈 cat internal/github.com/dilfish/inta/main.go
package inta

import (
    "github.com/dilfish/intc"
)

func P() {
    intc.P("from inta")
}
```
intb 的代码和 inta 类似，intc 的代码如下：
```
➜   gomodcrossref 😈 cat internal/github.com/dilfish/intc/main.go
package intc

import (
    "fmt"
)

func P(str string) {
    fmt.Println(str)
}
```

### 问题
此时用 IDE 打开 gomodcrossref 本身功能正常，但是打开 internal/github.com/dilfish/intb/main.go 就有问题了：
![如图所示](/pics/bad.ref.png)
internal/github.com/dilfish/inta/main.go 的情况是类似的，当然打开 intc/main.go 文件没有问题。

### 原因
原因很简单，就是 IDE 把 inta, intb 当成独立的模块，他的依赖要从对应的 go.mod 文件去找，默认不指定 replace 时，他直接从 github 网站找，那自然是找不到的，于是解析依赖失败。这个问题的诡异之处在于，项目依赖 inta, intb，所以他们必须是 module，而他们自己在 IDE 中要想走项目的总的 go.mod，又要求他们不能是 module，整个一个死循环。

### 方案
方案就是把缺失的 replace 加上。我看了 go mod 和 go list 的文档，第一个打算是写一个程序来处理每个包的依赖。 go list 有个命令：
```
go list -f '{{.Imports}}'
```
会把当前包依赖的包都打印出来，过滤标准库，剩下的就是需要 replace 的包，然后把需要 replace 的内容写进 go.mod 文件就好了。

实际运行发现不行，有些包的 go.mod 文件不一致，这个命令会失效，要求先运行 go mod tidy，这样得到的依赖包是空的。

后来又发现一个库是处理这类问题的：[packages](golang.org/x/tools/go/packages)，但我看了一下还挺复杂，而且我也不想花太多时间。

实在没辙，就把项目本身的 go.mod 文件里面的 replace 都给他复制进去，这样的话，inta 和 intb 的 go.mod 文件里会有许多多余的 replace 指令，go mod tidy 也不会去除，但是可以解决 IDE 的问题。如果要精准处理这个问题，可以先生成可用的 go.mod 文件，然后用程序去掉多余的 replace。因为 require 语句在 tidy 之后是没有多余的。

### 总结
这个坑总归还是自己的，正确的解法应该是按照 go 的标准，将 inta/intb 和 intc 分别作为两个 module，这样当 inta/intb 做好依赖管理之后，项目本身就不需要关心维护这个关系。

另外我发现一个事情，就算使用 replace, go get 依然访问 goproxy 去查看包的信息。这个协议的优先级还是以网络验证优先，本地维护在后。
