---
title: "Quic.Stream"
date: 2021-06-08T17:42:41+08:00
draft: false
---

### 概念
stream 是 quic 为上层应用提供的一个轻量级的有序字节流的抽象，可以是单向的，也可以是双向的。每个 stream 之间是独立的，所以不同 stream 之间并发的数据没有顺序保证。

### 类型
单向 stream 只能发送数据，双向 stream 可以发送和接收数据。stream 在 connection 之内是唯一的，由 stream ID 表示，他是一个 62 位的整数（0-2^62-1）。 同一个 connection 之内 stream ID 不能重复使用。

最低2位用来标识 stream 的类型，如下：
|  最后2位值   | 含义  |
|  ----  | ----  |
| 0x00  | 客户端发起，双向 |
| 0x01  | 服务端发起，双向 |
| 0x02  | 客户端发起，单向 |
| 0x03  | 服务端发起，单向 |

stream ID 是有序上升的，打开一个大的整数值（不包含最后2位）的 stream 意味着比他小的同类型 stream ID 全部打开。

### 收发数据
stream ID 和 offset 可以唯一确定一个数据，应用数据打包在 stream frame 里面。quic 可以接收多次同一个 stream ID + offset 的数据，但是如果数据有变化，则应该返回一个 PROTOCOL_VIOLATION 错误。

### 支持的操作

在发送端，stream 支持写入数据。以及关闭 stream (stream frame 设置 FIN 标记)和中断 stream (RESET_STREAM frame 如果 stream 在非关闭状态)

接收端的 stream 支持接收数据，以及终止 stream，通过发送一个 STOP_SENDING frame。

可选支持是 stream 状态变化，例如打开，关闭，等等。

### 状态机

发送和接受状态可以由状态机实现。
![send state machine](https://dev.ug/static.blog.dilfish.icu/send.state.machine.png)
新打开的 stream 处于 Ready 状态，此时 quic 可能会缓存住应用的数据。发送 STREAM 或者 STREAM_DATA_BLOCKED frame 之后，stream 进入 Send 状态。被动打开的 stream 收到对端打开 stream 的包之后，进入 Ready 状态。

拥塞控制由接收方控制，最多可以接收 MAX_STREAM_DATA 个 frame。如果发送的数据被拥塞控制算法阻止，则产生一个 STREAM_DATA_BLOCKED stream。

发送完数据之后，发送端会发送一个 STREAM frame 设置 FIN 标记，此时进入 Data Sent 状态，之后就只会重传丢失的数据，因此此时可以忽略拥塞算法（因为重传的请求都是对方发起的）。因为已经发出了 FIN 标记，所以此时还可能接收到 MAX_STREAM_DATA 个 frame。对端发送的 MAX_STREAM_DATA frame 可以安全地忽略。

当所有数据都被 ACK 之后，发送端进入 Data Recvd 状态，这是一个最终的状态。

正常状态（Ready, Sent, Data Sent）时，如果收到一个 STOP_SENDING frame，则会发送 RESET_STREAM frame，然后进入 Reset Sent 状态。当这个 RESET_STREAM frame 被 ACK 之后，则进入 Reset Recvd 状态，这是一个最终的状态。

![recv state machine](https://dev.ug/static.blog.dilfish.icu/recv.state.machine.png)

接收端的初始状态是 Recv，收到 STREAM, STREAM_DATA_BLOCKED, 或者 RESET_STREAM frame 都会创建接收端。双向的 stream 如果收到 MAX_STREAM_DATA 或者 STOP_SENDING frame 也会进入初始状态。

接收端发送 MAX_STREAM_DATA frame 让发送端可以发送更多数据。

收到 FIN 标记的 STREAM 之后，进入 Size Known 状态，此时不需要发送 MAX_STREAM_DATA，因为只可能重传数据而没有新数据了。

接收到全部数据之后，进入 Data Recvd 状态，应用层收到数据之后，进入 Data Read 状态，这是一个最终状态。

还没进入 Data Read 状态就收到 RESET_STREAM frame 会进入 Reset Recvd 状态，这时候可能数据都有，但是数据也有可能是不完整的。反之，如果进入 Data Read 状态再收到 RESET_STREAM frame，则可以安全忽略。

进入 Reset Recvd 状态之后，把状态发给应用，成功之后，就进入 Reset Read 状态，这是一个最终状态。

和状态有关的数据中，发送端有三种，STREAM，STREAM_DATA_BLOCKED 和 RESET_STREAM。而接收端只发送 MAX_STREAM_DATA 和 STOP_SENDING。

收到 STOP_SENDING 必须发送 RESET_STREAM，如果数据已经发完也可以等待结束，但是不能再重传数据了。
