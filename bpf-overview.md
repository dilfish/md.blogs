---
title: "Bpf 初探"
date: 2021-08-23T19:30:16+08:00
draft: false
---

## 概念

bpf 是 linux 下面的一个调试和跟踪的工具。依赖的技术包括之前零碎引进的 perf, kprobes, tracepoint 等等。这些工具提供了机制上面的追踪和修改功能，bpf 提供了一些策略上的常见的性能调优功能。因为这些工具是不同时间，不同方向引入的，所以对于特定的需求，可能有多个不同的实现方式。

## kprobe
kprobe 实现的原理是，传给他一个函数地址，他把对于地址置为中断指令，然后执行到那个地方的时候，在中断处理函数中调用用户提前注册好的处理函数。和调试器的原理有些像。

内核中的函数虚拟地址固定，而用户函数则复杂一些。注册了 uprobe 之后，用户打开可执行文件，uprobe hook 了执行可执行文件的系统函数，提前修改其在虚拟内存中的代码。其余原理和 kprobe 相同。

## tracepoint
tracepoint 以及其他静态的追踪功能实现都是在内核的函数里面添加 hooker 函数，再挂上一个用户注册的回调函数。和上述的 kprobe 原理上不同。

## 完整图
![图形](https://dev.ug/static.blog.dilfish.icu/bpftrace_probes.png)

## 参考文章
1. https://github.com/iovisor/bcc
2. https://jvns.ca/blog/2017/07/05/linux-tracing-systems/
3. https://www.douban.com/note/809579124/
