---
title: DevOps -- Global Scheduler
---

> Global Scheduler主要进行Pool级别资源分配。

## 主要职责

* 支持Pool注册/删除
* 支持Pool资源上报更新
* 支持基于自定义Rule规则的Pool分配
* 支持基于Pool Load / 客户端就近等原则的调度

## 调度Pool

调度过程分为Filter + Weight过程。

> 过滤Filter功能：
1) 去除不满足自定义规则的Pool
2) 去除Load过高的Pool

> 称重Weight功能：
1) 自定义规则Weight Pool
2) 客户端就近Pool
3) 按照Pool Load排序
