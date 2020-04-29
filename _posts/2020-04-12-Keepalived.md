---
title: DevOps -- keepalived Active Standby
---

> 一种通用的云上实现Active Standby的机制。

大概是： 利用Keepalived管理Active Standby的状态+Cloud LB管理入口VIP。

### 为什么需要？

主要是一些有状态的服务，如redis，pg，db等。自身不是很大的情况下，不适合做成cluster（cost）。但是又不想用Cloud Provider直接提供的服务，造成Lock in。所以有时需要自己搭建Active-Standby的集群。

### 有什么问题？

云上实现一个类似vip在主备机器上切换并不容易。
#### Option 1

腾讯/阿里有HAVIP专门做这样的事情。但是经我们在腾讯测试，发现了其在某些情况下，切换时间到达了分钟级别。原因是在云上的网络都是overlay的网络，底层是通过上层发送arp进行探测vip是否漂移的，每次vip的漂移都会导致底层路由更新。和腾讯负责这块的工程师一起debug过，极端情况下（例如：在上海区域这种有大vpc的情况），同步路由会有分钟级别的情况出现。 PS: 腾讯在我们测试后，紧急下线了这个功能。。。
阿里也差不多，不同的是，他之前主动下线了这个功能，要白名单才能使用，并且不带SLA。

#### Option 2

还有一种机制可以实现这个就是secondary ip。
每家云厂商都提供了secondary ip（包括aws），理论上，通过api接口detach/attach secondary ip，也是可以达到这种效果的，这种办法也类似于在Openstack里面使用allowed address pairs。 问题是，这种secondary ip往往比较难管理，我们通常以IoC的方式管理基础设施，那这些资源可能通过terraform生成。后期secondary ip detach/attach又上层应用控制，比较麻烦。


#### Option 3

目前我们使用的是折衷的方式，我们使用keepalived控制active - standby的状态（但是不由keepalived 控制vip），外加一个Cloud Load Balance。每个cloud 都有clb，这种方式可以通用，在云移植时，不用太多修改。
在私有化场景（物理机器），我们可以使用keepalived控制vip，这样在云上/非云环境配置几乎是一样的。
