---
title: "curl -X HEAD 问题"
date: 2021-04-16T18:15:12+08:00
draft: true
---

## 缘起
当你用 curl 测试服务的 HTTP 请求时，如果使用 -X HEAD 参数，curl 会警告说
`
Warning: Setting custom HTTP method to HEAD with -X/--request may not work the
`
`
Warning: way you want. Consider using -I/--head instead.
`
意思想要真正使用 HEAD 请求，应该使用 -I 或者 --head，而使用 -X HEAD 将可能不按你预期的方式工作。

## 测试结果
使用 -X HEAD 参数的结果是，返回了请求的 headers，并且 Content-Length 和普通的 GET 请求一样。但是请求没有完成，连接没有断开。

如果此时加上 -H 'Connection: close' 参数，则服务器会断开连接，而 curl 报错是 
`
curl: (18) transfer closed with x bytes remaining to read
`

说明正常情况下服务器遇到 HEAD 请求会停止发送数据，同时客户端断开连接。而 curl -X HEAD 只修改了请求参数，并没有处理 HEAD 方式的语义，导致连接没有断开。

用 nc 测试的话，连接不断可以继续发送请求，go 的 HTTP 服务端默认 Connection: keep-alive
