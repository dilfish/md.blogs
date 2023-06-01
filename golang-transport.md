---
title: "Golang Transport"
date: 2020-05-26T17:55:43+08:00
draft: false
---


最近我有一个需求，当发布了大量的域名资源到 CDN 时,需要去检测每个域名的资源是否预取成功。所以需要去 CDN 的每个边缘节点发送 HTTP 请求。特点是每次都是一个新的 Host。

其中有个优化问题，golang 实现 net/http.Transport 的时候，建议用户只使用默认的 DefaultTransport。因为他自己会启动2个 goroutine 去管理连接的读写。同时也有一个 map 去管理连接的缓存。

每一个正在请求的连接可以由调用的 Request 取消，而 idle 的连接则有2个函数可以控制。并且外部还有一个 Client.CloseIdleConnections 函数，其实现就是去调用 Transport 的对应函数。

这样当客户端使用时，对于 HTTP 协议，应该使用同一个 Transport ，因为他可以支持多个 goroutine 调用。并且在不断开链接的时候，可以更改 Host。

而 HTTPS 因为安全的缘故，禁用了这样的类似用法。对于 HTTPS 来说，Transport 会每次缓存链接，而如果每个请求都只是一个不同 Host 的资源的话，应该关闭 KeepAlive。这样连接关闭，和重新用一个 Transport 差不多。也用不上所谓 0-rtt 的 TLS 协议。这个问题要等到基于 quic 的 HTTP/3 才能解决。

HTTPS 不得不每个请求都使用一个新 Transport 就会导致自带的两个 goroutine 和 buffer 占用资源，当然关闭 Keep-Alive 可以很快关闭这些链接和 goroutine。golang 的垃圾回收对象不包括 goroutine。所以 Transport 关闭以后，看起来没有资源占用，实际上链接没断开的时候，goroutine 不会退出。

每次请求使用同一个 Transport 需要每次去修改他的 TLSConfig 对象，这个对象则是可以缓存的。而我们可以使用 Transport.MaxIdleConns 来调整缓存的连接数目。
