---
title: DevOps -- A failed connection caused by TCP MSS
---


> 今天在腾讯云环境发现从VPCB，无法访问VPCA中的Repo，但是Repo本身是好的，Security Group也没有问题。

## 问题现象

我们通过AWS北京区域，将产线的DC用IPSec连接成Hub - Spoke的内部网络，在腾讯云上海，我们有两个逻辑DC，分别用VPC A， VPC B与AWS北京DC连接。 AWS 北京DC作为IPSec Hub中心，用于转发各Spoke之间的流量。 今天问题出在在VPC B中一台VM，访问VPC A中的Repo时出错。经过调试Repo本身没有问题，ACL/Security Group也都没有问题。访问Repo时使用不同curl命令，有的成功，有的失败，例如：

```
# Failed
curl http://10.1.0.30/repo/rpms/qa/ms/centos7/x86_64/repodata/repomd.xml

# Success
curl http://10.1.0.30/repo/rpms/qa/ms/centos7/x86_64/
```





## 问题分析

最初怀疑Security Group，但很快排除，查看Repo的log，无异常。由于Repo部署在CLB --> Nignx 后面，所以查看Nginx，并无明显错误日志。 通过抓包，发现会有一些异常的ICMP:

![3](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/2019_11_05_3.png)

可以看到返回了需要分片，但是无法分片的错误。



查看IP报文头中，设定的是Dont Segment，不分片：

![1](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/2019_11_05_1.png)



TCP在协商过程中，会通过SYN包协商MSS值，用于决定PATH MTU的最大值：

![2](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/2019_11_05_2.png)

可以看到TCP连接的SYN包中，MSS是1424。 这是超过了IPSec的1400的长度的，而IP包头中，设定了DF（不要分片），因此出现了需要分片，但是不能分片的错误，导致了问题发生。



## 解决方法

在北京的VPN Server上，修改TCP SYN包中携带的MSS Option，将其修改为1360。 使用以下iptables命令在Mangle表中做如下动作：

  ```
# 配置mangle表，将1361-1536的MSS，统一改为1360.
#  iptables -t mangle -A FORWARD -o ens5  -p tcp -m tcp --tcp-flags SYN,RST SYN  -m tcpmss --mss 1361:1536  -j TCPMSS --set-mss 1360

# 保留配置，开机启动
# iptables-save > /etc/sysconfig/iptables 
  ```

追加此后，问题解决，可以正常访问。



注：

Linux中调整MSS还可以通过ip route在路由中做，本例中，使用iptables比较合适。 [2]中分析了内核MSS的动作，以及调整MSS的几种办法。



## Refer



[1] http://tools.ietf.org/html/rfc879#section-3

[2] https://paper.seebug.org/967/



