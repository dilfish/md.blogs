---
title: "tls1.3/http3 初探"
date: 2020-12-10T20:58:21+08:00
draft: false
---

上周有点忙，本来打算看完还是没看完。

curl 编译可以支持 http2/http3 tls1.3 :

https://github.com/curl/curl/blob/master/docs/HTTP3.md

cloudflare 的边缘服务器可以认为是一个标准的 http3 server。

nginx 怎么支持我还没看，好像 cloudflare 也发布了一个 patch。

https://blog.cloudflare.com/experiment-with-http-3-using-nginx-and-quiche/

HTTP 客户端发送请求时会带一个头叫 alpn:

like this:  ALPN: h2, http%2F1.1
https://httpwg.org/specs/rfc7639.html

但是目前 http3 还没有带，不知道是不是还没标准化的缘故。

这个 alpn 是放在 tls1.3 握手里面的，不在 HTTP 协议里。

同时，HTTP 服务端也有一个功能叫 Alt-Svc:

https://httpwg.org/specs/rfc7838.html

告诉客户端自己支持哪些协议。这个是 HTTP 头。

cloudflare 把服务端支持的协议打包放进了一个 DNS 记录里：

https://blog.cloudflare.com/speeding-up-https-and-http-3-negotiation-with-dns/

这样客户端在 tls 建立之前就可以决定使用哪个 HTTP 协议了。

golang 的 dns 库已经支持，我写了个客户端：

https://gist.github.com/dilfish/4f8ee232a7d0cec891933bd9b492505b

golang 库可以 mock 一个假的 listener 和 connection，打印 tls 协议细节：

https://gist.github.com/dilfish/f72e24c6ec2c6d5e1cd057f729b7c422

~~wireshark 怎么配置的还没测试好。~~

wireshark 可以配置 SSLKEYLOGFILE 这个环境变量，浏览器会把 tls 的 session key 写进去。之后就可以解码 HTTP3 的 tls1.3。解密出来 body 部分是文本，headers 部分使用了 qpack 压缩。是一个动态的霍夫曼编码算法。已经有代码可以解：

https://gist.github.com/dilfish/54437887807bf78475327b7381bc03c6

https://github.com/Zhiyi-Zhang/f-stack-for-quic

有一个 quic over dpdk 的方案，但是看起来只是打包组合起来。后续的 QoS 什么的都没有做很完整，不知道有没有更完整的方案开源出来。
