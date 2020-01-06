---
title: DevOps -- Nexus Repo Proxy
---

### Problem to solve

我们需要Docker Repo，Yum Repo，Java Maven Repo等多种Repo需求，Nexus[1]相当于一个AIO的Repo，他支持这些所有Repo。我们build了一个中心的Repo，在各个数据中心建立了Repo Proxy，用cache减少跨数据中心的流量（都是$$$）。


### Tips

目前我们使用nexus管理了rpm，maven，docker，搭建了proxy，集成了LDAP。目前没有太大的问题，以下是碰到的问题：
```
nexus3 proxyed repo 默认代理remote repo时。当发现remote 不存在某image，则本地repo会缓存远程不存在的结果，直到1440min后，再去尝试。
该配置可能是为减少不必要的remote call。

Negative Cache
Not found cache enabled: true (default)
Not found cache TTL: 1440 (default)

应该默认disable 该cache。 即local不存在，则一直向remote请求。
```


[1] https://help.sonatype.com/repomanager3
