---
title: DevOps -- packet drop 3
---

## Netfilter Conntrack溢出

在内核netfilter中，如果使用linux作为，比如网络中转节点，往往会保持非常多的conntrack。而默认的netfilter conntrack是65535，这个默认参数会使得连接较多时，无法建立新的连接。

出现这个问题时，会在日志中出现类似以下信息：
```
kernel: nf_conntrack: table full, dropping packet
```

解决：
提升nf_conntrack_max


建议监控：
监控nf_conntrack_count的变化，对nf_conntrack_count / nf_conntrack_max的百分比进行告警。


## 虚拟设备队列


vether pair/tap等虚拟队列长度。


## Socket半连接队列


半连接队列，linux默认值是128。 对于压力较大的HTTP/TCP服务器，这个值可以优化。

```
/proc/sys/net/core/somaxconn
```


## TCP/UDP Buffer


对于高性能tcp/udp服务器，往往需要更多的tcp/udp buffer

### 解决办法

加大buffer:
```
sysctl -w net.core.rmem_max=8388608
sysctl -w net.core.wmem_max=8388608
sysctl -w net.core.rmem_default=65536
sysctl -w net.core.wmem_default=65536
sysctl -w net.ipv4.tcp_rmem='4096 87380 8388608'
sysctl -w net.ipv4.tcp_wmem='4096 65536 8388608'
sysctl -w net.ipv4.tcp_mem='8388608 8388608 8388608'
sysctl -w net.ipv4.route.flush=1
```


### 监控告警

netstat -s
prometheus
