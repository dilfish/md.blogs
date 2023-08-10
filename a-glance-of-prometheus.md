---
title: "Prometheus 初探"
date: 2023-07-19T19:39:19+08:00
draft: false
---

### 起因
好像几年前随着 k8s 兴起，prometheus 就流行了，我最近才开始接触。粗略地过一遍。

### 概念
- target: 目标机器
- job: 监控任务，和上面的机器是不同维度的数据，加上时间，就变三维数据了
- metric: 在 job 下面的细分，具体的监控对象
- exporter: prometheus 是主动拉取(scrape)监控数据的，被拉取对象叫 exporter，输出数据，一般在 /metrics
- scrape_interval: 抓取间隔，所以数据结果是离散的点，太快了会不会把服务器打爆了啊..


### metrics
- count: 计数器，单调递增的，和时间一起，就是一个斜线，斜率变化则表示该指标增长速率有变化
- gauge: 单个指标数量，均匀的数量应该是一个直线
- histogram: 分区间统计

如图：
![我的 grafana](https://dev.ug/static.blog.dilfish.icu/prom.png)
