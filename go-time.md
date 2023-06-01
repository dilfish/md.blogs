---
title: "Go 时间"
date: 2022-07-29T15:19:49+08:00
draft: false
---

### 起因
最近 facebook 发了 [一个文章](https://engineering.fb.com/2022/07/25/production-engineering/its-time-to-leave-the-leap-second-in-the-past/)，大概意思是说，闰秒给程序员带来了很多麻烦，希望可以取消。

### 计时类型
[这个网站](http://www.leapsecond.com/java/gpsclock.htm) 列出了常用的几个计时方式。其中 UTC 是程序界比较常用的。他的秒的时长是按原子钟计时的，但是时间数会和格林威治时间进行校准。而格林威治时间是根据地球的转动情况设置的，所以 UTC 时间并不均匀，有可能加速也有可能减速。闰秒就是按秒调整，其他的方案比如 facebook 的 smearing 就是把这个加速减速给平摊到更长的时间区间内。

GPS 和 TAI 的计时是从0开始严格均匀递增，秒的长度是按原子钟，所以没有闰的概念。但是程序界没有普遍使用这两个计时方式。

### 程序支持
linux 从 3.10 之后支持了 TAI 计时，clock_gettime 有一个 CLOCK_TAI 选项。示例程序的结果是：
```bash
CLOCK_REALTIME : 1659076964.254 (19202 days +  6h 42m 44s)
CLOCK_TAI      : 1659077001.254 (19202 days +  6h 43m 21s)
CLOCK_MONOTONIC:    2497325.086 (28 days + 21h 42m  5s)
CLOCK_BOOTTIME :    2497325.086 (28 days + 21h 42m  5s)
```

可以看到 CLOCK_REALTIME 也就是 wall clock，就是我们常用的 UTC 时间，由 ntp 进行调整。而 TAI 时间我还没看是怎么校准的。(反正挺复杂 [linux 代码](https://github.com/torvalds/linux/blob/master/kernel/time/timekeeping.c))

CLOCK_MONOTONIC 和 CLOCK_BOOTTIME 就是机器的 uptime。

[go tai](https://github.com/brandondube/tai) 的实现是先获取 UTC，然后加上和 TAI 的差值。但这个实际上是一个虚假的 TAI 计时，如果不更新库代码，TAI 计时依然有可能出现倒退。

[go](https://pkg.go.dev/time) 采用了一个很聪明的做法，他在显示端依然用 UTC，但是在计算端，例如 Before After 或者 Sub Add 同时维护了另一个单调递增的时间，取自 CLOCK_MONOTONIC。所以不会再出现 cloudflare 那个 bug 了。因此他不需要操作系统支持 CLOCK_TAI，代码里 linux 版本也未使用。

MacOS 不但有 CLOCK_MONOTONIC，还有 CLOCK_MONOTONIC_RAW，后者不受 CPU 频率或者时间调整的影响。不知道前者是什么情况。
