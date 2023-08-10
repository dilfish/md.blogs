---
title: "白嫖 vps"
date: 2023-06-06T23:50:52+08:00
draft: false
---

最近偶然间看到一个 azure 的学生免费政策，开始尝试了几种免费的云主机。
    
大概可以表述为，如果你只用 aws 和 azure，他们各有一个免费一年的 vps，那么申请3张双币卡，有效期六年，就可以无限白嫖下去。

有点出乎意料的是，通常日本节点都比香港节点要快，大约日本实力强，所以海底光缆近吧。

另外一点是，aws 和 azure 都设计了一套庞大而精细的计费策略，比国内的几个厂要先进一些，这个和工业品的规模优势的道理差不多。

另外还测试了 quic 和 tcp 的优劣，实测下来，quic 延迟低，tcp 带宽高。但是 google 大规模推广 quic 可能是因为美国人民有钱，网络中间的盒子更标准吧，能把 quic 跑到 tcp 差不多的带宽。

fastly 的[这篇文章](https://www.fastly.com/blog/measuring-quic-vs-tcp-computational-efficiency)总结得很全面。

vmess 不层层加密的话，正常下载10m文件消耗11.4m流量，有效数据比还是比较高的。
