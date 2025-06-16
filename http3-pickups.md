---
title: "Http3 拾遗"
date: 2025-06-16T19:56:54+08:00
draft: false
---

## DNS 的 HTTPS 类型
HTTPS 类型最特别的是他有一个 host 部分，我研究了一下发现，这个部分实际上是为了和 HTTP Header 的 Alt-Svc 配合的，例如请求 a.com 的 https 类型，他可以返回 www.a.com 和 m.a.com的结果。增加了许多灵活性。

而且这个名称也有格式，例如 SVCB 记录里的名字是 _8443._https.api.example.com，实际上指导了端口和协议信息。旨在一次性包含 HTTP 请求的所有信息。

## HTTP 版本降级
我给某个子域名开启了 HTTP3 功能，其他因为兼容选择不开，但是 nginx 目前还不支持这样的配置。问了 deepseek 说的是，好几种办法，例如 QUIC 协议层拒绝，客户端会自动尝试 HTTP2。还有优雅降级是返回 421 状态码。还有一个情况，不返回 UDP 包，客户端会超时重试 HTTP2，这比较显然了。
