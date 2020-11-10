---
title: WebRTC -- RTT
---

> 本文描述WebRTC是如何计算RTT的。

LSR: last SR，上一个SR发送时间戳， 32bit取了秒/毫秒的精度。
DLSR: Delay of last SR，处理时延。

RTT时间为： 当前时间 - LSR - DLSR

备注：
1. 当前时间 和 LSR在同一台机器，无需担心时间不准，我们仅看差值。
2. DLSR是接收端处理总时长。
