---
title: DevOps -- PanoStack Spec
---



**私有化PanoStack提供以下三种模式：**

1） PanoStack单机版：         适用于中小客户，可以支持音频/视频的基础功能，默认支持Windows/Mac/IOS/Android端，可选支持白板/Web/录制/直播推流。并发用户可以支持10-400路

2）PanoStack HA版 ：          适用于中大客户，对HA要求高，功能与单机版类似，可以支持高可用Failover，并发用户数可以水平扩展。

3）PanoStack 全功能版本： 适用于大客，需要Case By Case商务洽谈，拥有完整功能包括数据分析，多DC，加速，日志分析，更细致告警。





**功能对比如下：**

| 功能点            | PanoStack单机版 | PanoStack HA版 | PanoStack全功能版 |
| ----------------- | --------------- | -------------- | ----------------- |
| 支持APP数         | 1个             | 1个            | 可支持多个        |
| 音频通话          | 支持            | 支持           | 支持              |
| 视频通话          | 支持            | 支持           | 支持              |
| Web客户端         | 可选            | 可选           | 支持              |
| 白板              | 可选            | 可选           | 支持              |
| 录制              | 可选            | 可选           | 支持              |
| 直播推流          | 可选            | 可选           | 支持              |
| 监控              | 基础            | 基础           | 全面              |
| 告警              | 基础            | 基础           | 全面              |
| HA高可用/Failover | 不支持          | 支持           | 支持              |
| 并发容量          | 低              | 中             | 高                |
| 多DC              | 不支持          | 不支持         | 支持              |
| 海外加速          | 不支持          | 不支持         | 支持              |
| 日志分析          | 不支持          | 不支持         | 支持              |
| 数据罗盘          | 不支持          | 不支持         | 支持              |
| 计费              | 不支持          | 不支持         | 支持              |