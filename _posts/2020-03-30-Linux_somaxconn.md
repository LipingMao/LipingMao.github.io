---
title: DevOps -- tcp半连接和全连接队列
---

Linux在处理TCP新建连接时有两个队列：
 * 半连接队列
 * 全连接队列



我们TCP建立连接有三次握手：

 * Client ----SYN----> Server
 * Client <--SYN/ACK-- Server
 * Client ----ACK----> Server



对于Server端，当接收到SYN请求（第一个包）后，这个请求就进入了半连接队列。当接收到Client的ACK后（第三个包），这个请求就进入了连接队列。当服务器调用accept方法，这个请求就从连接队列中取出，进行处理。



看一下关于这两个队列相关的内核参数：

```
net.ipv4.tcp_max_syn_backlog = 256
net.core.somaxconn = 128`
```

默认的半连接队列长度为256， 全连接队列为128。 一般可以优化为半连接2048，全连接1024以上。
