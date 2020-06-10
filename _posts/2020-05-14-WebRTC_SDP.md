---
title: WebRTC -- SDP
---



> 记录和学习一些基本的音视频知识。

SDP（Session Description Protocal）说直白点就是用文本描述的各端（PC 端、Mac 端、Android 端、iOS 端等）的能力。

主要包括 SDP 描述格式和 SDP 结构，而 SDP 结构由会话描述和媒体信息描述两个部分组成。SDP 是由一个会话层和多个媒体层组成的；而对于每个媒体层，WebRTC 又将其细划为四部分，即媒体流、网络描述、安全描述和服务质量描述。

[RFC4566](https://tools.ietf.org/html/rfc4566#page-24)
