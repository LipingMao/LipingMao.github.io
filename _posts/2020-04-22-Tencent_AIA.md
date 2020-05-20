---
title: DevOps -- Tencent AIA
---


> 最近发现Tencent AIA在海外加速方面对比专线有很大优势。本文大概介绍AIA。



Anycast 公网加速（Anycast Internet Acceleration，AIA）是一个覆盖全球的动态加速网络，可以大幅提升您业务的公网访问体验。AIA 不同于其他应用层加速服务，它能实现 IP 传输的质量优化和多入口就近接入，减少网络传输的抖动、丢包，最终提升云上应用的服务质量，扩大服务范围，精简后端部署。



Anycast 利用了BGP在不同的地区的对外宣告相同IP，从而达到IP可以被client端就近访问到。

当客户从就近的Pop点接入后，所有的网络都走内网，相当于达到了内网专线的效果：

![image-20200520212308928](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/image-20200520212308928.png)



相当于借助腾讯Backbone实现了全球加速的能力。