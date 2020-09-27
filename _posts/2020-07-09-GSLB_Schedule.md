---
title: DevOps -- GSLB Two Layer Schedule
---



当前实现GSLB和SLB在调度时，部分节点在GSLB调度(例如：MS/RTMS/RTC GW/RN)，部分节点在SLB调度(例如：BS/TS)。目前的这种实现，今后DC规模扩展会存在问题，在支持Cascading时也会存在很多状态需要跨DC同步的问题。



最初设计时GSLB / SLB时，是按照以下职责设计的：

```
GSLB： 负责DC级别负载调度，Channel与DC绑定关系。
SLB ： 负责节点负载调度，Channel与节点绑定关系。
```

希望达到的设计容量是： 1024 DC * 1024 Nodes(上限）



最初这个设计客户端请求分配节点的流程是：

```
客户端请求GSLB分配DC， GSLB分配SLB。

客户端请求SLB分配节点，SLB分配节点。
```



后来，当初讨论为了做到“减少客户端与Pano集群的交互次数，希望客户端一次请求就直接获取节点”。

于是，SLB将MS节点的信息注册/上报到了GSLB。

后来支持了RTMS节点，就将RTMS也注册到了GSLB。

之后支持了海外加速RN，RN节点的信息也上报到了GSLB。

再之后支持了WebRTC GW，这类节点信息也上报到了GSLB。

目前除了TS/BS，录制/推流时在SLB做调度，其他的服务器都放在了GSLB调度。





这么做存在以下**风险点**：

1. 与最初的设计初衷不符合，越来越多的节点角色被GSLB管理，GSLB需要知道节点的实时状态，负载情况。这些节点情况需要SLB上报到中心。

* 每个Node的Load/状态发生变化时，都需要从SLB同步到GSLB。这导致设计DC * Nodes的数据需要上报到一处，也就是理论上限 1024 * 1024个节点的数据需要准实时同步到GSLB。

* SLB到GSLB之间往往时跨DC的，跨DC的网络有一定延迟，跨DC的情况下保持数据一致性会相对困难。



2. 后面需要支持节点的Cascading的调度，在分配Cascading时，需要同时考虑节点Load，和每个channel在每个节点上有多少人的信息。换句话说：

* 每个channel在每个媒体节点上的人数发生变化时，都需要同步到GSLB，需要同步的数据量是与Channel数目 * Cascading的节点数。



这两点都会造成中心GSLB处理太多DC内部细节信息，成为系统分布式扩展的瓶颈。理想情况下，只有在增加新DC的时候，SLB应该仅有有限的数据与GSLB同步（与节点个数无关/与channel数目无关），这样架构上比较容易扩展。



**回到我们的设计上限目标是：**
GSLB 可以管理1024个DC
SLB    可以管理1024个Node

假设每个Node上面有50个channel，理论上会有50 * 1024 * 1024个channel信息。



GSLB应该仅关心1024个DC状态和整体load，以及50*1024*1024的channel信息与DC之间的关系。 但是不会有channel级别的信息与SLB之间应该仅有Event代表channel开始或者结束。

SLB应该仅关心本DC的1024个节点的状态和load，以及50*1024个channel与Node的绑定关系，以及这些channel/user级别信息(用于决策cascading)。

这样不管是SLB 还是 GSLB在分配节点时，仅关心大概1000个左右的分配对象。对于Channel，仅同步开始/结束事件。