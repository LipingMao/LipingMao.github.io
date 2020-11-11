---
title: WebRTC -- jitter
---

A 到 B发送数据:
TA1: 数据1发送时间
TA2: 数据2发送时间
TB1: 数据1接收时间
TB2: 数据2接收时间

抖动(j) = | (TB2 - TA2) - (TB1 - TA1) |

实际抖动会进行平滑
