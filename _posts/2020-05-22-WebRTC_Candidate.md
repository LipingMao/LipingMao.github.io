---
title: WebRTC -- Candidate
---


> 什么是 ICE Candidate

它表示 WebRTC 与远端通信时使用的协议、IP 地址和端口，一般由以下字段组成：
```
{
  IP: xxx.xxx.xxx.xxx,
  port: number,
  type: host/srflx/relay,
  priority: number,
  protocol: UDP/TCP,
  usernameFragment: string
  ...
}
```

其中，候选者类型中的 host 表示本机候选者，srflx 表示内网主机映射的外网的地址和端口，relay 表示中继候选者。

WebRTC 会按照上面描述的格式对候选者进行排序，然后按优先级从高到低的顺序进行连通性测试，当连通性测试成功后，通信的双方就建立起了连接。在众多候选者中，host 类型的候选者优先级是最高的。在 WebRTC 中，首先对 host 类型的候选者进行连通性检测，如果它们之间可以互通，则直接建立连接。其实，host 类型之间的连通性检测就是内网之间的连通性检测。WebRTC 就是通过这种方式巧妙地解决了大家认为很困难的问题。同样的道理，如果 host 类型候选者之间无法建立连接，那么 WebRTC 则会尝试次优先级的候选者，即 srflx 类型的候选者。也就是尝试让通信双方直接通过 P2P 进行连接，如果连接成功就使用 P2P 传输数据；如果失败，就最后尝试使用 relay 方式建立连接。
