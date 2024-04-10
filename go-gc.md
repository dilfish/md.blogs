---
title: "Go 的垃圾回收算法"
date: 2022-07-25T18:07:01+08:00
draft: false
---

### 起因

本文是对文档 https://go.dev/doc/gc-guide 的阅读整理。

### 介绍
第一部分比较简单，介绍了 gc 的缘由等。其中 object 是一片内存，包含了一个或者多个 go 的值。pointer 是一片内存，内容是对某个 go 值的引用。 go 的 gc 算法是 mark-sweep 算法，并且是 non-moving 的。

### gc cycle
mark 阶段和 sweep 阶段必须是严格分开的，不能并行，因为 mark 没有完成之前，sweep 不能确认某个内存是否被其他内存引用，因而不能回收。

gc 消耗的四个事实：

- gc 执行的时候，应用程序暂停
- gc 只消耗 CPU 和内存
- gc 的内存消耗包括 live 内存，mark 之前申请的 heap 内存，以及元数据
- CPU 的消耗和 live 内存成比例

如果要定量分析 gc 的话，可以从最简单的情形开始，也就是所谓的 steady-state。

- 应用程序请求内存的速度固定，byte/second。这个申请速度和内存是否 live 是无关的，即申请了可以立即释放，变成 dead 状态，也可以一直不释放而保持 live 状态。当然也可以边申请边释放，通常来说 live 内存有一个最大值，而且这个最大值小于等于申请的总量。
- 应用程序的对象图每次都一样。如果 gc 算法是一个固定的算法的话，每个相同的请求肯定会产生相同的对象树。

这个模型虽然比较简单，但是可以用微积分的思路去理解，也就是说，真实的程序可能是这个模型的一个傅里叶级数。

如果 gc 算法提早运行，那么 CPU 消耗会增加，而 live 内存的峰值会降低。反之亦然。这是一个 CPU 和内存的 trade-off。GOGC 环境变量来控制这个频率，可以设置 GOGC=100，或者 GOGC=off。

### GOGC
GOGC 的值代表的是将要运行 gc  之前能申请的内存的值和已经使用内存的值的比值。已经申请内存包括三部分，一是栈空间，二是全局变量，三是 live heap。

这个定义的好处是，让 gc 的消耗保持固定。让 gc 的频率和需要做的工作保持固定比率。如果在 steady-state 之后，快速申请内存，那么 gc 就会更频繁运行，而如果在 steady-state 之后缓慢申请内存，那 gc 的频率会降低。

如果是上述的固定频率的请求，那么 gc 的运行频率也是固定的。

互动动画显示，在应用使用内存固定频率的情况下， gogc 设置越大，则 gc 运行越不频繁，因而内存峰值越高，但是 CPU 使用越低。

而当 go 运行在有限内存的机器上时，有可能 gc 不得不更频繁地运行，以确保内存不会超过内存限制。

### memory limit
go 在 1.19 里面添加了一个 memory limit 的选项，主要是为了适应容器化的部署。

![短暂峰值](https://blog.871116.xyz/pics/transient.peak.png)

上图展示了一个内存的短暂的峰值，如果没有这个峰值，GOOS 应该可以设置比如200，因为通常的 live heap 是 20MB，而总的内存是60MB。但是这个短暂的峰值 live heap 达到了 30MB，所以 GOOS 最多只能设置 100.

使用全局变量 GOMEMLIMIT 或者 runtime/debug.SetMemoryLimit 可以设置内存的上限。设置了内存上限之后，无论 GOOS 设置多大或者是 off，go gc 不会使用超过限制的内存。但是如果应用程序使用了很多内存，那么代价就是 gc 更频繁地运行。

### Optimization guide
- inuse_objects—Breaks down the number of objects that are live.
- inuse_space—Breaks down live objects by how much memory they use in bytes.
- alloc_objects—Breaks down the number of objects that have been allocated since the Go program began executing.
- alloc_space—Breaks down the total amount of memory allocated since the Go program began executing.
