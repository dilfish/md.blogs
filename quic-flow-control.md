---
title: "Quic 拥塞控制"
date: 2021-06-09T19:59:16+08:00
draft: false
---

### 介绍
本章只描述了一个整体的框架，具体算法可以由实现自由发挥

### 拥塞控制
quic 只控制数据流，握手协议不做控制，实现应该提供一个握手协议包控制的方式。单个 stream 和整体的 stream 都有相应的控制方法，同时，stream 个数也是有限制的。

在握手阶段，接收者会发送他能接受的整体的最大字节量。然后，可以发送 MAX_STREAM_DATA(单个 stream) 或者 MAX_DATA (所有 stream)frame 来增加这个值，如果是减少的将被忽略。

当发送者被上面两个指标限制，可以发送 STREAM_DATA_BLOCKED 或者 DATA_BLOCKED 告诉接受者应该增加这个值，如果长期不增加，那么发送者可能会关闭这个 stream 或者 connection。但是接受者不能等待这两个 frame 来做拥塞控制，因为这不是必须发送的。

最多可以打开 max_streams * 4 + first_stream_id_of_type 个 stream。

### 下集预告
本章比较短，内容也只是描述框架。但是下一个 connection 的概念比较复杂，包含如下章节：
- connections
- version negotiation
- cryptographic and transport handshake
- address validation
- connection migration
- connection termination
- error handling
