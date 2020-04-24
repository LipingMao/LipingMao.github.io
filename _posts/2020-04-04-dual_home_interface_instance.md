---
title: DevOps -- Dual home interface Instance
---

今天帮忙看一个多网卡的问题，如果一个主机多网卡时，需要进行路由表划分。否则会出现路由问题，最常见的是只能访问一个接口的ip。（因为linux会检查入包和出包是否同源）。 AWS上创建多接口的Instance的路由表如下：

```
[root@ip-10-242-81-39 rc.d]# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 9001
        inet 10.242.81.39  netmask 255.255.255.128  broadcast 10.242.81.127
        inet6 fe80::55:52ff:fe19:e859  prefixlen 64  scopeid 0x20<link>
        ether 02:55:52:19:e8:59  txqueuelen 1000  (Ethernet)
        RX packets 1221  bytes 116604 (113.8 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1583  bytes 148132 (144.6 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eth1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 9001
        inet 100.64.0.130  netmask 255.255.255.0  broadcast 100.64.0.255
        inet6 fe80::70:7aff:fe35:5a5f  prefixlen 64  scopeid 0x20<link>
        ether 02:70:7a:35:5a:5f  txqueuelen 1000  (Ethernet)
        RX packets 1257  bytes 104188 (101.7 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1228  bytes 167899 (163.9 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 8  bytes 648 (648.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 8  bytes 648 (648.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

可以看到有两个IP地址。此时可以配置ip rule, 如果是从eth1来的包，则查看路由表10001：
```
[root@ip-10-242-81-39 rc.d]# ip rule
0:	from all lookup local
32765:	from 100.64.0.130 lookup 10001
32766:	from all lookup main
32767:	from all lookup default
```

分别看一下路由表，可以看到不同表上默认路由不同：
```
[root@ip-10-242-81-39 rc.d]# ip route show table 10001
default via 100.64.0.1 dev eth1
100.64.0.0/24 dev eth1 proto kernel scope link src 100.64.0.130
[root@ip-10-242-81-39 rc.d]# ip route show table main
default via 10.242.81.1 dev eth0
default via 100.64.0.1 dev eth1 metric 10001
10.242.81.0/25 dev eth0 proto kernel scope link src 10.242.81.39
100.64.0.0/24 dev eth1 proto kernel scope link src 100.64.0.130
169.254.169.254 dev eth0
```
