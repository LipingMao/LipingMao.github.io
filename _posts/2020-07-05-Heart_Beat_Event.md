---
title: DevOps -- Heartbeat and Event
---

### 心跳

> Scheduler/Node间需要同步Load状态

Pool Scheduler 和 Node之间保持节点的Load信息。

Global Scheduler 和 Pool Scheduler之间保持Pool/Group的Load汇总信息。


### 通知

> 客户端加入/离开时，会上报状态信息用于绑定一组客户端信息。


例如： 一组客户端需要被调度到同一个服务器时。

Global Scheduler需要维护这一组客户端与DC的绑定关系。
Pool Scheduler需要维护这一组客户端与Node的绑定关系。
