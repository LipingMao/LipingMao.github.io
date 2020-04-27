---
title: DevOps -- Ali Cloud active-standby vip
---

> 本文描述阿里云上如何支持Active-Standby VIP.

以下可能的方式：
*  阿里支持HAVIP，不过目前已经下线，需要白名单支持。
*  使用阿里的Active-Standby的SLB。
*  使用ENI的Secondary IP。


ENI的Secondary IP的方式参考[1],[2].
大致思路是：
```
keepalived管理vip迁移。
在变成Master时，将对端vip去除，设置vip到本地。
在变成Slave时， 将本地vip去除，设置vip到对端。
```

[1] https://help.aliyun.com/document_detail/101180.html?spm=5176.2020520101.0.0.12704df5idYkZF
[2] https://help.aliyun.com/document_detail/101263.html?spm=a2c4g.11186623.6.929.7d0c4c57V2BXWN
