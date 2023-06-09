---
title: "golang 的 gc"
date: 2021-12-06T22:55:53+08:00
draft: false
---

## 概念

go 是我用的第一个使用 gc 的语言，之前对这一块不是很熟悉。这几天看了很多相关的文章，有一些大概的了解。gc 和调度以及内存分配有比较亲密的关系。新版的 go 使用的内存分配在向 tcmalloc 靠近。而调度则是抢占式的时间片分配。是用信号机制实现的。

## 三色标记法

三色标记法算法的证明还比较复杂，我也不想看。总之当前使用的是 Dijkstra 1987年的一个论文和 Yuasa 1990 年的一个论文。叙述起来他是一个比较保守的标记算法，每一步执行过程中，标记的是一定指向其他对象的对象。因此，被标记的是不能回收的对象，当然算法有可能把可以回收的也标记了。那么剩下的肯定没有被其他对象指向的对象，则保证可以回收。

### writer barrier

这里的 wb 和 linux 中通常针对 CPU 的 wb 不同。而是一个更大尺度的 wb，也可以认为是当 gc 和 app 交替运行过程中，确保算法的内存状态一直的那么一个机制。这里的 wb 是一段代码，新版的 go 将这段代码写在编译器里面。

因为 wb 打开的时候，每个指针写操作都要跑一次 wb 代码，所以这个性能肯定很闹心，也提了很多优化的办法，批量写啊，缓存啊等等。但是 go 的目标是把这个操作和 app 并发跑起来，因而会让 app 正常跑得慢一些，而 gc 本身不显得非常慢。

最近有一个 issue 提到要增加 gc 的 pace，即让这个过程更加频繁。说不定以后可以做到极致就是完全同步跑，不再分 gc 和非 gc 过程。这样就会变成一个每一步都稍慢一些的程序而没有明显的 gc 停顿。

### 阅读
https://draveness.me/golang/docs/part3-runtime/ch07-memory/golang-garbage-collector/
这个老哥有毅力，看了很多代码和文章。
