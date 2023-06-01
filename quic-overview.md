---
title: "Quic 协议概览"
date: 2021-05-31T19:23:53+08:00
draft: false
---

### stream
stream 是对字节流的抽象，保证对于上层应用来说，是一个有序的字节流。stream 有四种，分别是客户端和服务端发起的，单向和双向的 stream。

stream 的接受和发送方，各维持一个状态机，如果是双向的 stream，则对发送和接受各维持一个状态机。

打开 stream 只需要给对方发送已给新的 stream id 就可以，关闭 stream 则可以正常关闭，或者中断 stream。

stream 的流控也比较简单，针对单个 stream，接收方可以随时指定他的新的可接受的 offset。

### connection

connection 需要握手才能建立，同时依赖底层的加密协议验证对端。并发或者不同优先级的数据只需要在 stream 层面控制即可。connection 层面主要和下层的 UDP 协议合作，提供一个对端的抽象。即，对端变换了 ip 和 port，可以新建一个 connection，而依然走 特定的提前商议好的秘钥。

多个 connection 也可以起到加密的作用，比如使用不同的 connection id 发送数据，可以降低中间人攻击的概率。

### packet and frame

数据的载体是 packet，packet 是被加密过的数据单位。放在 UDP 报文中。

packet 内容是 frame，协议的类型主要放在 frame 中，例如 PING, ACK, RESET 等。

### 整体
整体上来看，frame 负责传输应用数据和命令控制。packet 负责加密内容，不同等级的 packet 可能会使用不同等级的加密层级。

connection 负责维护两边短点，例如地址变换，混淆数据等。而 stream 负责一个有序的字节流，例如 http 中的某个文件。其中，拥塞控制实现在 stream 和 connection 层中，例如某个优先级高的文件的传输速度可以更快。或者某个 ip 的数据传输更快。connection 机制给了不同 ip 使用同一个 quic 加密提供了条件。

