---
title: DevOps -- Keepalived Failover Issue
---

> 本文记录了keepalived在腾讯云中时碰到的问题。

我们的问题是在HK DC，使用了腾讯云的HAVIP提供Active Standby的HA。
一开始处于以下状态：
VM1 : Master
VM2 : Slave

发生问题时，VM2因为一些原因，切换成了Master，马上又切换会了Slave，此时VM2发出了一次garp。
此时，腾讯云底层实际是overlay网络，这时他会将vip的路由指到了VM2。
然而，对于keepalived来说VM1还是master，VM2是slave，因此，此刻开始Keepalived和腾讯云就有了不同的状态。

解决这个问题可以给keepalived添加以下配置：
```
vrrp_garp_master_refresh    10  // 如果我是master，我每10秒钟对外发送免费arp
vrrp_garp_lower_prio_repeat 2   // 如果我是master，但我收到了低priority的instance尝试变成master，则发送2次免费arp
vrrp_garp_lower_prio_delay  1   // 延迟1s发送第二组
```

有了以上机制之后，master总是会产生garp，不管底层是否绑定正确，最终一定会将底层状态转为一致。
