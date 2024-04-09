---
title: "DNSPod 审计"
date: 2023-07-28T21:42:02+08:00
draft: false
---

### 起因
在做 CDN 调度的时候，我设置了一个辅助域名，用来判断 DNS 解析将某个 IP 划分到哪个区域。例如使用 dns subnet 选项，测试 1.1.1.1 这个客户端属于 cloudflare，那我就在配置时选择将 cloudflare 区域的 TXT 记录配置为 cloudflare 这个字符串。将所有支持的区域都配置为对应的名称之后，就可以通过 TXT 请求来判断某个 IP 的归属了。

### 代替 IP 库？
商业的 IP 库还挺贵的，好像一年最少好几万。ipip 已经卖到十几万了。那么将 IPv4 空间的 IP 全部请求一遍，就得到了一份 DNS 服务商的 IP 库，之前读 RFC 的时候我就意识到了这个问题，但是一直没有实践。

测试发现一个比较麻烦的问题，IP 的区段通常是不规则的，例如 0xffff_0013 - 0xffff_0028 属于上海电信。但是 DNS subnet 协议里面使用了 cidr，这就划分成了好几段。所以扫完 IP 空间还需要更久一点，估计得一两天吧。所以精准度上商业的 IP 库确实比 DNSPod 还是高一些。但是如果节省成本并忍受一点低精度，这个确实还是不少钱。

### 不同实现
RFC7871 里面有建议：
`
To protect users' privacy, Recursive Resolvers are strongly encouraged to conceal part of the user's IP address by truncating IPv4 addresses to 24 bits. 56 bits are recommended for IPv6, based on [RFC6177].
`

我测试了下，google 严格遵守了这个建议，腾讯没有遵守，可见隐私国内确实也不值钱。

cloudflare 实现了这个功能，但是不允许指定 IP，只能使用真实的 IP，这应该是一个更好的方案。

从这些区别中也能看来，这个 DNS 调度的精度比302调度还是差了不老少。隐私和精度是互斥的。

### 计划
等我有时间，对比一下 DNSPod 的 IP 库和商业库，看看精准度，打通一下代码。
