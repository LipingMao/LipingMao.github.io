---
title: WebRTC -- Candidate
---


WebRTC 将 Candidate 分为三种类型：
```
host 类型，即本机内网的 IP 和端口；
srflx 类型, 即本机 NAT 映射后的外网的 IP 和端口；
relay 类型，即中继服务器的 IP 和端口。
```

其中，host 类型优先级最高，srflx 次之，relay 最低。

