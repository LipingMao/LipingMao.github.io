---
title: AWS -- NAT Gateway cost
---

## Background

> When we use AWS VPC, a common practice is to build public subnetwork and private subnetwork, and typically, you need to deploy AWS NAT Gateway for your private subnetwork. NAT Gateway is AWS managed , HA, Auto-scale gateway. But when I check the billing dashboard, I am supprised by the cost of NAT gateway, it is even more expansive than one t3.xlarge + one t3.medium RI instances. It will cost 0.37 RMB per hour in NingXia(China) region. T3.xlarge is 0.2465 RMB per hour, T3.medium is 0.0616 RMB per hour. So we decide to use one medium bastion server work as nat gateway in our case. In this blog, I am talking about how to set up NAT Gateway in your bastion server.

## Steps

Here is step by step install guide:
1. Disable source/dest
![Disable Source Dest](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/2019_08_11_1.png)

2. Enable NAT on NAT VM server:
```
# Enable ip_forward
vim /etc/sysctl.conf
net.ipv4.ip_forward = 1
# Enable sysctl
sysctl -p
# Add NAT rule:
iptables -t nat -A POSTROUTING  -s 10.0.4.0/24 -j MASQUERADE
service iptables save
```

3. Update private subnetwork route table:
![Update private network route table](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/2019_08_11_2.png)
