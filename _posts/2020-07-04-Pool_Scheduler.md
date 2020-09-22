---
title: DevOps -- Pool Scheduler
---

> Pool Scheduler主要进行Pool级别资源分配。

## 主要职责

* 支持Node注册/删除
* 支持Node资源上报更新
* 支持基于自定义Rule规则的Node分配
* 支持基于Node Load / Node group的调度

## 调度Pool

调度过程分为Filter + Weight过程。

> 过滤Filter功能：
1) 去除不满足自定义规则的Node
2) 去除Load过高的Node
3) 去除非本Group的节点

> 称重Weight功能：
1) 自定义规则Weight Node
2) 按照Node Load排序
