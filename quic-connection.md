---
title: "Quic 连接"
date: 2021-06-16T19:57:31+08:00
draft: false
---

### connection
connection 是客户端和服务端共享的一个状态。每个 connection 建立的时候都需要握手，通过握手生成一个秘钥，商议上层应用使用的参数，以及本 connection 的参数。0-RTT 是客户端在服务端响应之前就发送数据，服务端也可以在收到最后一个握手信息之前发送信息，但是安全性会降低，这给了应用层一个选择安全性和性能平衡的能力。

每一个 connection 都有一个 connection ID 的集合，其中每一个都可以代表这个 connection. connection ID 由端点独立选择，每个端点选择对端使用的 connection ID.

因为 connection ID 是接收端选的，所以他可以适应下层的地址变化，例如 IP 或者端口变化。收到数据后依照 connection ID 进行验证。

空的 connection ID 虽然支持，但是会失去上述的迁移功能。

每个 connection ID 还会附加一个序号，以区分 NEW_CONNECTION_ID 和 RETIRE_CONNECTION_ID frame。最初的 connection ID 是放在握手时 long packet header 的 source connection id 区域。此时序号是0, 如果有 preferred_address 参数，那序号是1.

额外的 connection ID 通过 NEW_CONNECTION_ID frame 下发，每次序号必须加一。一旦下发之后，端点必须接受这个 connection ID 发来的数据，直到 connection 结束或者对端发来 RETIRE_CONNECTION_ID frame 停止。 connection ID 不能高于对端提供的 active_connection_id_limit 值。如果临时超过，必须提供足够大的 Retire Prior To 值提示对端退休掉一部分 connection ID. 如果退休后还多，那么返回一个 CONNECTION_ID_LIMIT_ERROR 错误并关闭整个 connection。

进行迁移时，端点必须确保对端还有未用的 connection ID,不然对端就没法确认迁移结果了，因为迁移需要使用新的 connection ID。迁移过后，之前的地址使用的 connection ID 应该退休。

packet 到 connection 的匹配，如果校验出错那么包会被丢弃。

空的 connection ID 可以认为是本地专用的。

### version negotiation

如果服务端收到一个不支持的版本，他应该发送一个 Version Negotiation packet.

如果服务端收到一个没有版本信息的 packet，例如  Initial packet，那他会自己选择一个版本.

客户端如果发来一个服务端不支持的版本，服务端可能会发一个 Version Negotiation packet  回复。前提是客户端发来的数据包足够大，可以让服务端选择多个版本中的一个。

如果客户端收到一个不支持的  Version Negotiation packet ，他应该关闭 connection。除非之前已经成功收到了一个，或者选择的版本就是他指定的那个。

CRYPTO frame 来发送加密握手, 当前的版本号是 0x00000001，使用 TLS，详细内容在 rfc9001。

加密握手必须实现几个功能：
- 验证秘钥交换
  - 服务端是必须验证的
  - 客户端是可选验证的
  - 每个 connection 都产生一个独立的秘钥
  - 秘钥可以用作0RTT和普通的1RTT包
-  可以保护传输参数，服务端的参数必须加密
-  应用层协议保护

![simple quic handshake](https://dev.ug/static.blog.dilfish/simple.quic.handshake.png)
带*的地方是可以发送应用数据的地方。

![1rtt quic handshake](https://dev.ug/static.blog.dilfish/1rtt.quic.handshake.png)
第一个包是 Initial 类型，packet 号是0，包含了一个 CRYPTO frame，内容是 ClientHello.
多个不同类型的 packet 可以放在一个 UDP 包里面，所以这个握手需要最少4个 UDP 包。

![0rtt quic handshake](https://dev.ug/static.blog.dilfish/0rtt.quic.handshake.png)
connection ID 是为了保护路由的一致性，long header 包含2个 connection ID，dest connection ID 是接收端选择的，src connection ID 是为了对端设置 dest conntion ID用的.

握手过程中，每个端点都会选择 src connection ID，让对方可以发送给自己数据。在第一个 Initial packet 之后，每个端点再发送数据时都需要设置 dest connection ID。

客户端第一次发送 Initial packet 时，可以选择一个至少8字节的 dest connection ID 临时使用，在收到服务端返回的真正的 dest connection ID 之前，它必须一直使用这个 ID. 同时设置自己的 src connection ID。0rtt 包需要使用的 src 和 dest connection ID 与客户端第一次建立连接时使用的一样。收到服务端的 Initial packet 之后，就确定了 dest connection ID。唯一可变的只有服务端的 NEW_CONNECTION_ID frame 了。

实现至少要有能力缓存4096字节的加密包。
### address validation

地址验证就是为了防止放大攻击的。在验证地址之前，服务端最多发送3倍客户端发来的数据。
通常情况下，握手协议可以达到地址验证的目的。

验证之前，端点发送的数据不能大于收到数据的三倍，连接建立和迁移的时候都要进行地址验证。加密握手可以认为是地址验证成功了，因为有密钥和 conn ID

为了防止死锁，客户端要加超时，同时尽可能使用大包。服务端可以用 token 先验证客户端，再建立连接。服务端可以多发额外的 token 给客户端迁移地址的能力路径验证只验证发送方向，返回方向由对方验证


### connection migration

握手期间不能变换地址，也可以指定参数允许或者不允许迁移。

客户端主动发起迁移，服务端不能迁移。如果客户端发起的新地址不可用，服务端还使用原来的地址。
新的地址的拥塞控制参数可能会变化。
服务端主要防止客户端的各种虚拟地址攻击。
