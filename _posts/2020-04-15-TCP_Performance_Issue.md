---
title: DevOps -- TCP Slow Start
---


> 最近在做程序性能测试时发现一个POST请求花了100+ms的时间。本文记录了debug过程和一些tcp的相关内容。


为了简化说明这个，我们可以将问题抽象如下：
* 有一个HTTP服务器，提供一个Handler是POST请求，这个POST请求里面，可以认为什么都不做，只是读取客户端的Request Body就直接返回了。
* HTTP客户端发送了一个POST Request， Request Body大概是500KB左右。
*  客户端和服务器处在两个数据中心，之间的时延大概在20ms左右。

![image-20200502222141095](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/image-20200502222141095.png)

**问题是：**明明在Handler里面除了打印一下request body什么也没做，但是Handler却花了100ms左右。

那这100ms究竟花在了哪里呢？Wireshark抓包看一下：

![image-20200502224108340](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/image-20200502224108340.png)



从Wireshark抓包中可以看到：
10.0.0.83 是客户端IP，10.0.4.166是服务器端IP， 可以发现每隔一段时间，发送端会出现一次0.02s左右的时延。一共出现了5次，因此有了100+ms的时延。



分析TCP的Seq Num可以明显看到这种等待趋势，可以看到明显的step：

![image-20200502232910232](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/image-20200502232910232.png)

图中是tcp seq num增长曲线。从这张图，看到TCP的Seq Num有明显的stepping，每次stepping都等待了20ms左右，这个时间正是一个RTT的时间。



![image-20200504210247811](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/image-20200504210247811.png)

再看TCP Window Scaling，可以看到窗口每经过1个RTT就大概扩大1倍。而每次RTT之后window scaling一倍，正是TCP慢启动过程的典型动作（慢启动过程中每次RTT会增加一倍的window size）。



快速复习一下TCP的慢启动过程：

TCP在三次握手之后，便会发送数据包，但是此时TCP并不清楚客户端和服务器端的时延/带宽状况，他应该发送多少数据比较合理呢？因此TCP有个探测网络的过程，这个过程会采用慢启动策略。这个策略非常简单，每次收到服务器端的ACK时，就将滑动窗口设置为上一次的两倍。假设初始窗口大小为X，然后发送端会发送X个字节，当发送端收到服务端的ACK时，发送端就会把窗口大小设置为2倍，然后继续发送，下次再收到ACK时，就将发送窗口提高到4倍。保持这种指数增长直到达到ssthresh。



找到了问题，解决这个问题，一般有两种思路：
* 短连接改为长连接。 如果改为tcp连接，那不需要每次都建立连接，这样就没有慢启动过程。
* 通过调整初始initcwnd/initrwnd(修改TCP的初始窗口大小），以及接收/发送端的tcp send/receive buffer大小，从而提高跨数据中心/高延迟的tcp传输性能。


Key take aways：
* Wireshark分析TCP问题时，可以借助tcptrace查看是否有stepping存在，如果存在stepping，一般都有tcp buffer问题/tcp窗口大小相关的问题。
* Linux 默认配置对于相同数据中心的TCP连接来说，没有太大问题，因为DC内部的时延通常都在0.X ms。但对于时延较大时，相应参数需要调整，否则可能存在性能问题。
* tcp短连接存在慢启动限制，如果传输body不大的情况下，没太大感知，但是如果body又比较大，又是短连接，此时传输效率不高。



