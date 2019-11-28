---
title: DevOps  -- Keepalived in Docker Tips
---



当Keepalived 1.3.+跑在Docker中时，会碰到以下问题：

```
IPVS: Protocol not available
```



这是因为Keepalived1.3+ 需要在宿主机器上加载ip_vs， 简单workaround可以在宿主机器上modprobe ip_vs:

```
# lsmod | grep ip_
  ip_vs_wlc 16384 3
  ip_vs 151552 5 ip_vs_wlc
  nf_conntrack 131072 1 ip_vs
  libcrc32c 16384 3 nf_conntrack,xfs,ip_vs
```



确认ip_vs加载后keepalived可以启动成功。



另一个解决办法是：

其实我们在使用keepalived时并没用用lvs的功能，可以将keepalived升级至2.X，使用2.X版本后，启动时对于lvs是没有依赖的。



[1] https://bugs.launchpad.net/ubuntu/+source/keepalived/+bug/1800159
