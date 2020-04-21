---
title: DevOps -- TCP Slow Start
---

对于TCP短连接来说，很大程度上性能会被TCP慢启动影响， 一下是一个tcptrace的图：

![image-20200422074702398](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/image-20200422074702398.png)



可以看到在途中有若干处，发送方等待接收方发送了ACK之后才继续发送，等待时间接近RTT时间。

通过修改初始windows size可以扩大初始tcp windows：

```
# 接收方
ip route change default via x.x.x.x initrwnd 20

# 发送方
ip route change default via x.x.x.x initcwnd 20
```





[1] https://blog.habets.se/2011/10/Optimizing-TCP-slow-start.html
