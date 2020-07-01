---
title: AWS -- Network Cost
---

AWS EC2流量收费按照以下集中模式（不同Region收费不同，以中国区为例）：

*  EC2流量流出到公网（非AWS Region），以宁夏/北京为例，费用是0.933元/GB

* EC2流量流出到AWS其他Region，以宁夏/北京为例，费用是0.6003元/GB
* EC2在同一个Reion不同AZ中流出流量，以宁夏/北京为例，费用是0.067元/GB
* EC2在同AZ中使用公网地址通信，以宁夏/北京为例，费用是0.067元/GB
* EC2在同一个AZ使用内网地址通信，不收费。





https://datapath.io/resources/blog/what-are-aws-data-transfer-costs-and-how-to-minimize-them/