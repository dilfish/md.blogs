---
title: "Rust Atomics and Locks 书评"
date: 2023-03-06T16:51:11+08:00
draft: false
---

### 起因
来自友邻的推荐：https://book.douban.com/subject/35904892/

### 内容
着重看了第三章和第七章。 rust 的内存模型比 go 语言的更底层一些，但是又比 c/c++ 的高一些。

其中基于硬件的原因因为涉及到 CPU 实现，无法精确地复现。只能说指令重排和缓存一致性的影响。

文章介绍了 lock 指令，mfence 相关指令，缓存一致性算法等。以及编译器和 CPU 对于程序指令的重排。

内存模型总结起来可以分为四个类型：
- 没有原子操作，这种类型的数据多线程访问会导致未定义行为
- Relaxed 内存顺序，这种数据只适合做计数器，总数不会错，但是每个线程获取到的值不能用作其他验证
- Release/Acquire，这个顺序在使用者的两个线程间建立了一个 happens before 关系，但是其他线程看来，数据依然不确定
- SeqCst 全局一致顺序

和数据库的事务模型有点像。

go 语言只有最后一种类型，所以不需要关心这类问题。

四种内存一致性其实反应了四种不同的硬件指令配置，但是具体反应的问题是什么无法一一对应。
例如 Release 顺序，应该是 sfence或者干脆 mfence，或者是 store buffer 的写入。这个无法确定。
![如图](/pics/mem.order.jpg)

### 参考资料
这方面参考资料简直太多了：
- https://darkcoding.net/software/rust-atomics-on-x86/
- https://lwn.net/Articles/250967/
- https://research.swtch.com/mm
- https://go.dev/ref/mem

### 八卦
这个作者挺有意思，人称大胸姐，twitter 签名里 tag 是 ADHD, Polyamorous, Lesbian
