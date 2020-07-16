---
title: DevOps -- Redis Multi Master
---

### 问题内容：

新建DC时，每次Redis都出现多主。



### 问题原因：

是因为先部署了Keepalived，后部署Redis。 Keepalived先选主完成，Redis在部署过程中会重启，在重启时，会丢失主备信息，造成与Keepalived状态不一致。
如果单独重启备份Redis，就可以再现这个问题。



### 如何解决：

```
1. 重启Redis时，sleep 5s，超过Keepalived的检测时间，让keepalived检测到这个状态。
2. Keepalived判断自己的状态与Redis状态不一致时，进入fault状态。
```
