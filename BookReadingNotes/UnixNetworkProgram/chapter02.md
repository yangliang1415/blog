# 传输层: TCP、UDP和SCTP
> yangliang@20161001



## 概述及各协议简介
本章的焦点是 `传输层`，包括TCP、UDP、SCTP，如下

* UDP: `不可靠` 的 `数据报` 协议。不保证UDP数据报一定可达；不保证数据报先后有序；不保证每个数据只到达一次
* TCP: `可靠` 的 `字节流` 协议。提供客户与服务器之间的 `链接`；提供 `可靠性`(数据的可靠投递或故障的可靠通知)；提供 `流量控制`；提供确认、序列号、RTT估算、超时和重传等机制；`全双工`
* SCTP: 暂未了解



## TCP连接建立和终止
本小节我们了解TCP连接是如何建立和终止的，并掌握TCP状态转换图，以后就能更好的理解connect、accept和close等网络编程中遇到的函数。

###连接建立: 三次握手
服务器准备好接受外来的连接。一般通过调用socket、bind、listen这3个函数完成，称作 `被动打开`。然后客户端发起TCP连接，建立TCP连接一般如下三个阶段

* 1: client通过调用connect发起 `主动打开`。这导致client端TCP发送一个SYN同步分节。
* 2: server端必须ACK确认client端的SYN，自己也发送一个SYN分节
* 3: client端再确认server端的SYN，发送ACK

### TCP选项
TCP连接的一些参数设置

###连接终止: 四次挥手



## TCP的TIME_WAIT状态
2个原因

* 最终的ACK丢失
* 延迟连接化身的新建: 防止来自某个连接的老的重复分组在该连接已终止后再现，从而被误解成属于同一个连接的某个新的化身。


## TCP端口号和并发服务



## 缓冲区大小及限制



## Ref
* http://blog.csdn.net/whuslei/article/details/6667471
* http://www.jianshu.com/p/21b5cbac0849

