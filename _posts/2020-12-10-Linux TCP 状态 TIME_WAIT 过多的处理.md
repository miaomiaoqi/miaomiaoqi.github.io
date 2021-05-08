---
layout: post
title: Linux TCP 状态 TIME_WAIT 过多的处理
categories: [Network]
description: 
keywords: 
---


* content
{:toc}




首先处理这个问题, 我们要知道一些网络知识, 要知道tcp那些事, 比如说三次握手, 和四次挥手......很多人会问, 为什么建链接要3次握手, 断链接需要4次挥手? 让我们一起看下下面的流程图: 

![http://www.milky.show/images/net/timewait/t_1.png](http://www.milky.show/images/net/timewait/t_1.png)

首先, 是三次握手: 

首先Client端发送连接请求报文, Server段接受连接后回复ACK报文, 并为这次连接分配资源。Client端接收到ACK报文后也向Server段发生ACK报文, 并分配资源, 这样TCP连接就建立了。

然后是中间部分:  **两者之间可以传输数据了**

再次, 下面的断开链接:**[注意]中断连接端可以是Client端, 也可以是Server端。**

假设Client端发起中断连接请求, 也就是发送FIN报文。Server端接到FIN报文后, 意思是说"我Client端没有数据要发给你了", 但是如果你还有数据没有发送完成, 则不必急着关闭Socket, 可以继续发送数据。

所以你先发送ACK, "告诉Client端, 你的请求我收到了, 但是我还没准备好, 请继续你等我的消息"。这个时候Client端就进入FIN_WAIT状态, 继续等待Server端的FIN报文。当Server端确定数据已发送完成, 则向Client端发送FIN报文, "告诉Client端, 好了, 我这边数据发完了, 准备好关闭连接了"。

Client端收到FIN报文后, "就知道可以关闭连接了, 但是他还是不相信网络, 怕Server端不知道要关闭, 所以发送ACK后进入TIME_WAIT状态, 如果Server端没有收到ACK则可以重传。“, Server端收到ACK后, "就知道可以断开连接了"。Client端等待了2MSL后依然没有收到回复, 则证明Server端已正常关闭, 那好, 我Client端也可以关闭连接了。Ok, TCP连接就这样关闭了！

那么可以这么理解, 当client进入time_wait的等待时间是2个MSL

让我们看一下一台linux服务器的网络状态: 

```bash
# netstat -an | awk '/^tcp/ {++State[$NF]}END{for(key in State)print key "\t" State[key]}'LAST_ACK  7
LISTEN  9
SYN_RECV  2
CLOSE_WAIT  125
ESTABLISHED  1070
FIN_WAIT1  17
FIN_WAIT2  247
CLOSING  4
TIME_WAIT  25087
```

对于网站来说, 这样的time_wait略显偏高, 也就是说大量的关闭操作在等待2个MSL后结束, 正常我们的tcp 端口是65535个, 如果并发再高一些, 可能会大量的socket不能及时被释放, 从而导致性能下降, 所以我们可以通过linux内核进行一些网络调整比如, 开启socket重用和快速回收: 

```bash
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_max_tw_buckets = 5000
net.ipv4.tcp_max_syn_backlog = 8192
net.ipv4.tcp_keepalive_time = 1200
net.ipv4.ip_local_port_range = 1024 65000
```

**net.ipv4.tcp_syncookies = 1**

表示开启SYN Cookies。当出现SYN等待队列溢出时, 启用cookies来处理, 可防范少量SYN攻击, 默认为0, 表示关闭；

**net.ipv4.tcp_tw_reuse = 1**

表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接, 默认为0, 表示关闭；

**net.ipv4.tcp_tw_recycle = 1**

表示开启TCP连接中TIME-WAIT sockets的快速回收, 默认为0, 表示关闭。

系统tcp_timestamps缺省就是开启的, 所以当tcp_tw_recycle被开启后, 实际上这种行为就被激活了.如果服务器身处NAT环境, 安全起见, 通常要禁止tcp_tw_recycle, 至于TIME_WAIT连接过多的问题, 可以通过激活tcp_tw_reuse来缓解。

**net.ipv4.tcp_max_tw_buckets = 5000**

表示系统同时保持TIME_WAIT套接字的最大数量, 如果超过这个数字, TIME_WAIT套接字将立刻被清除并打印警告信息。默认为180000, 改为 5000。对于Apache、Nginx等服务器, 上几行的参数可以很好地减少TIME_WAIT套接字数量, 但是对于Squid, 效果却不大。此项参数可以控制TIME_WAIT套接字的最大数量, 避免Squid服务器被大量的TIME_WAIT套接字拖死。

**net.ipv4.tcp_max_syn_backlog = 8192**

表示SYN队列的长度, 默认为1024, 加大队列长度为8192, 可以容纳更多等待连接的网络连接数。

**net.ipv4.tcp_keepalive_time = 1200**

表示当keepalive起用的时候, TCP发送keepalive消息的频度。缺省是2小时, 改为20分钟。

**net.ipv4.ip_local_port_range = 1024-65000**

表示用于向外连接的端口范围。缺省情况下很小: 32768到61000, 改为1024到65000。

```bash
# netstat -an | awk '/^tcp/ {++State[$NF]}END{for(key in State)print key "\t" State[key]}'
LAST_ACK	140
LISTEN	9
SYN_RECV	7
CLOSE_WAIT	2
ESTABLISHED	972
FIN_WAIT1	21
FIN_WAIT2	152
CLOSING	2
TIME_WAIT	682
```

