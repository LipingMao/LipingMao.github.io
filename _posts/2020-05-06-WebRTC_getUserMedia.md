---
title: WebRTC -- Overview
---



> 记录和学习一些基本的音视频知识。



Google推出了的WebRTC使得浏览器+WebRTC就可以实现实时音视频通信。一个最基本的一对一音视频通信过程如下：

![image-20200530200236684](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/image-20200530200236684.png)

大致分为几个部分，WebRTC终端，信令服务器，STUN/TURN服务器。

* WebRTC终端负责音视频采集/编码/NAT穿越/数据传输等。
* 信令服务器负责信令处理，如加入房间/离开房间/媒体协商消息等。
* STUN/TURN负责NAT打洞穿越/数据中转。



一次WebRTC通信大致过程如下：

WebRTC一端进入房间之前，检查本地设备情况并进行音视频数据采集，之后与信令服务器通信，发出加入房间。另一端做类似的事情，信令服务器负责User List管理，对端用户会收到另一个用户加入的消息。之后WebRTC终端通过RTCPeerConnection进行通信。期间会尝试P2P打洞，如果成功则直接和对端传输数据，如果失败则通过TURN服务器转发数据。