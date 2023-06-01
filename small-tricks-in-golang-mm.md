---
title: "golang 内存模型里的小细节"
date: 2021-09-17T18:32:11+08:00
draft: false
---

## 起因

最近看完了 Russ Cox 写的那个[ Memory Models ](https://research.swtch.com/mm)老三篇，顺便又仔细看了一遍 go 官方的[那个文档](https://golang.org/ref/mem)。

## 问题一
官方文档里面有这么一句话：`A receive from an unbuffered channel happens before the send on that channel completes.`不仔细看会以为这个话的意思是管道的接收操作发生在发送操作之前。实际上这句话里面 completes 是形容 send 的，意思是说，接收操作发生在发送完成之前。

也就是说，在没接收的时候，发送操作是不能完成（返回）的。

同样的，对应的那句是：`A send on a channel happens before the corresponding receive from that channel completes.`意思是发送操作发生在接收完成之前，也就是说，没有发送操作，接收操作是不能完成的（这个很显然）。而上面那个命题就必须加一个 unbuffered 限定条件。

也就是说，他用两个偏序关系定义了一个相等/同时关系。

可能相等的描述起来会比较麻烦，如果操作是时间点还好说，如果是时间段的话，那这个同时的意思是，同时结束。

#### update 20220208

看看下面这个程序输出：
```go
package main

import (
    "log"
    "time"
)

func main() {
    ch := make(chan struct{})
    go func() {
        time.Sleep(time.Second)
        close(ch)
    }()
    ch <- struct{}{}
    log.Println("good closing")
}
```

```bash
➜   testch 😈 ./testch
panic: send on closed channel

goroutine 1 [running]:
main.main()
	/Users/dilfish/go/src/testch/main.go:14 +0x7e
```

## 问题二
sync.RWMutex，这个属于之前没有仔细看。
`If a goroutine holds a RWMutex for reading and another goroutine might call Lock, no goroutine should expect to be able to acquire a read lock until the initial read lock is released. In particular, this prohibits recursive read locking. This is to ensure that the lock eventually becomes available; a blocked Lock call excludes new readers from acquiring the lock.`
这个意思是说写锁的优先级高于读锁。

## 计划
后续打算看看这个长文[ Getting to Go: The Journey of Go's Garbage Collector](https://go.dev/blog/ismmkeynote)
