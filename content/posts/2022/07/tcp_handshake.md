---
title: "TCP为什么需要三次握手"
date: 2022-07-17T20:41:32+08:00
draft: false
author: "chimission"
categories: ["network"]
tags: ["tcp"]
archives: ['2022', '2022-07']
comments: false
description: "tcp三次握手"
url: "/posts/tcp_handshake/"
---
>你真的知道tcp为什么需要握手3次吗
 <!--more-->

TCP协议作为网络传输的基础协议之一，学过计算机的同学都知道一条基本的原则 -- `TCP 协议建立连接需要经过双方三次握手（handshake）`。 

相信大家也或多或少看到过下面这个类比，来解释为什么TCP需要3次握手来建立连接 
>1 你能听见我吗？  
 2 我能听见你，你能听见我吗？  
 3 我能听见你  

我在刚开始学习计算机网络的时候也觉得这样能够很好的解释为什么TCP需要3次握手来建立连接，但是这种类比的问题在于，把复杂的问题简单化了，这虽然有助于我们快速理解相关概念，但是缺点是丢失了上下文，把问题需要注意的关键细节隐藏了起来。当我们想要深入挖掘问题的本质时，类比就无能为力了。

### 连接是什么
解答为什么之前首先我们需要先明确一下什么是`连接`。

很多TCP协议的教学文章都没有明确对`连接`给出一个解释，[RFC 793 - Transmission Control Protocol ](https://en.wikipedia.org/wiki/Handshaking)给出了对`连接`的明确定义：
>Connections:
The reliability and flow control mechanisms described above require
that TCPs initialize and maintain certain status information for
each data stream.  The combination of this information, including
sockets, sequence numbers, and window sizes, is called a connection.
Each connection is uniquely specified by a pair of sockets
identifying its two sides.

即`连接`是数据结构，这些数据结构用来保证TCP通信的可靠性和流控制机制（Socket、序列号以及窗口大小）。建立`连接`实际上就是初始化这些数据结构的过程。`连接`完成就是初始化完数据结构，两端就可以依靠这些数据结构记录的信息进行数据传输了。 

### 三次握手
让我们来看一下`连接`建立的过程，即三次握手初始化相关数据结构的过程。

1. 首先`客户端`发起连接请求，发送一个 TCP 的 SYN 标志位置1的包(SYN=1, seq=x)，指明`客户端`要连接的`服务端`端口，以及初始序号 X,保存在包头的序列号(Sequence Number)字段里。
2. `服务端`发回确认包(SYN=1, ACK=1, seq=y, ACKnum=x+1)应答。即 SYN 标志位和 ACK 标志位均为1。`服务端`生成自己 ISN 序列号，放到 Seq 域里，同时将确认序号(Acknowledgement Number)设置为`客户端`的 ISN 加1，即X+1。 发送完毕后，`服务端`进入 SYN_RCVD 状态。
3. `客户端`再次发送确认包(ACK=1，ACKnum=y+1)，SYN 标志位为0，ACK 标志位为1，并且把`服务端`发来 ACK 的序号字段+1，放在确定字段中发送给对方，并且在数据段放写ISN的+1。发送完毕后，`客户端`进入 ESTABLISHED 状态，当`服务端`接收到这个包时，也进入 ESTABLISHED 状态，TCP 握手结束， `连接`建立。  
通过上边总结的过程我们知道了TCP连接过程主要是两端共同协商初始化了各自的数据结构，包括创建socket，确定seq，ack值等等。

### 两次握手
想象一下，如果TCP使用的是两次握手会出现什么情况？
在我们回顾TCP三次握手过程中，并没有考虑网络拥堵状况，如果在握手过程中网络状况复杂或者比较差，`客户端`发送的请求包会滞留在网络中`服务端`收不到请求也就不会有回复，从而引起`客户端`超时重试。现在问题来了，网络中存在两个`客户端`请求包，会前后分别到达`服务端`，如果TCP只有两次通信的机会，那么服务端只能选择接受或者拒绝请求，因为`服务端`没有请求包的上下文。
所以TCP需要三次通信来让`客户端`确认`服务端`接受到的请求是有效的，当`服务端`收到了请求包时，`服务端`会将请求包里的 seq 加1 返回给`客户端`（对应在三次握手的第二步），由`客户端`来判断此次请求是否有效。

如果当前连接是历史连接，即 seq 过期或者超时，那么`客户端`就会直接发送 rst 控制消息中止这一次连接；
如果当前连接是有效的，那么`客户端`就会发送 ack 控制消息，通信双方就会成功建立连接；
`服务端`会根据接收到的消息作出不同的处理。

### 总结
[RFC 793 - Transmission Control Protocol](https://datatracker.ietf.org/doc/html/rfc793)里也指出了TCP 连接三次握手的主要原因 —— 为了阻止旧的的重复连接初始化造成的混乱问题，防止使用 TCP 协议通信的双方建立了错误的连接。
>The principle reason for the three-way handshake is to prevent old duplicate connection initiations from causing confusion.

至于为什么是3次而不是4次5次，这是因为 TCP 消息头的设计，TCP可以将中间的两次通信合成一个同时发送 ACK 和 SYN 控制消息，进而减少一次通信成本，讨论使用更多的通信次数来建立连接是没有意义的，因为我们总可以使用更多的通信次数交换相同的信息，所以使用四次、五次或者更多次数建立连接在技术上是完全可以实现的，为了节省成本，当然是越少的通信次数越好。
