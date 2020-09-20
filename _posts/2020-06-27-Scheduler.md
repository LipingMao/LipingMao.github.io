---
title: DevOps -- Scheduler Overview
---

# 调度系统架构

> SaaS服务/PaaS服务的调度系统主要负责将工作负载平衡到各个工作节点。比如：音视频的服务，通常需要有SFU媒体服务器作为工作节点实际处理媒体流转发，就需要解决客户端与媒体服务器之间调度的问题。


## 总体架构

对于大型系统，通常将调度分成多层结构，有一个全局的调度服务负责将负载调度到自治域中(通常是数据中心/虚拟数据中心/Pool集群)。在自治域中负责将负载分配到实际工作节点。 因此从调度角度，有以下两个组件：

* Global Scheduler
* Pool   Scheduler 


整个系统的调度由以下组成：
1 * Global Scheduler + N * Pool Scheduler

客户端请求获取工作服务器流程如下：
```
1. Client请求Global Scheduler, Global Scheduler调度Pool Scheduler，返回Pool Scheduler给Client。
2. Client请求Pool Scheduler获取工作服务器。
3. Client连接工作服务器。
```


### Global Scheduler

通常来说Global Scheduler仅负责Pool级别的调度， Pool需要定期将状态同步到Global Scheduler。

Pool Scheduler心跳上报的信息可以是：

```
Load : 用于描述本Pool负载情况，可以分不同维度的Metrics，也可以是归一化的数据（例如将不同维度的负载情况，归一化到0-1000之间）。

Status : 表示Pool当前状态是否可以被分配，用于区分服务正常 / 处于维护状态 / 处于未知状态等。

Group : Pool可以对服务器再划分资源组，用于实现资源优先级/VIP资源隔离等能力。

Location : 地理位置等信息用于判断就近接入。

Capability : 如果各个Pool有能力区别，可以用于上报支持什么能力。
```

有了这些上报的信息，Global Scheduler就可以根据各个Pool的负载情况将Client调度到各个Pool。同时Client在请求Global Scheduler时，Global Scheduler可以获取到Client的Public IP，可以根据Public IP进行基于地理位置的调度。将Client就近调度到一个DC中。


### Pool Scheduler 

Pool Scheduler负责节点级别的调度, 每个工作节点定期将工作负载，运行状态等相应信息上报给Pool Scheduler。

工作节点心跳上报的信息可以是：
```
Type : 节点类型，表示不同的能力。

Load : 节点的Load情况(可以是综合了CPU/Memory/Network等归一化之后的数据)

Status : 表示节点当前状态是否可以被分配，用于区分服务正常 / 处于维护状态 / 处于未知状态等。

Group : 节点属于哪个资源组。
```

Pool Scheduler可以使用上述信息调度客户端到不同的工作节点。
