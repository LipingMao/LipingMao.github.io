---
title: WebRTC -- RTCP
---



> 记录和学习一些基本的音视频知识。



在使用 RTP 包传输数据时，难免会发生丢包、乱序、抖动等问题，下面我们来看一下使用的网络一般都会在什么情况下出现问题：

* 网络线路质量问题引起丢包率高；
* 传输的数据超过了带宽的负载引起的丢包问题；
* 信号干扰（信号弱）引起的丢包问题；
* 跨运营商引入的丢包问题 ;
* ……



如何知道这些网络质量信息？RTCP可以帮助获取这些信息，RTCP 有两个最重要的报文：RR（Reciever Report）和 SR(Sender Report)。通过这两个报文的交换，各端就知道自己的网络质量到底如何了。

以SR为例，头部如下：

![image-20200531231851849](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/image-20200531231851849.png)

RTCP头部含义：

![image-20200531231903917](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/image-20200531231903917.png)