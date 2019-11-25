---
title: DevOps  -- Keepalived On Different Cloud Tips
---



> 当在云上使用keepalived做Active-Standby控制VIP时，通常需要在云上配置“允许”VIP的动作。本文记录了OpenStack，AWS，Tencent Cloud不同的允许VIP的方式。



## Openstack

Openstack提供了Allowed Address Pairs的方式打开Anti-IP/Mac-Spoofing的限制。默认情况下OpenStack不允许非Neutron分配的地址被分配到虚拟机上。[1]



## AWS

在AWS上，VIP漂移时需要使用AWS的API做associate EIP和Private IP，见[2] [3]。 当然此处也可以使用secondary IP，但是也是需要调用AWS API做Private IP的迁移的。



## Tencent

在腾讯云上，有内测的HAVIP功能，申请了HAVIP后，VM是会allow HAVIP的地址发出来的，相当于这个租户所有机器开了allow address pair这个HAVIP。





[1] https://blog.codecentric.de/en/2016/11/highly-available-vips-openstack-vms-vrrp/

[2] https://www.peternijssen.nl/high-availability-haproxy-keepalived-aws/

[3] https://icicimov.github.io/blog/high-availability/Keepalived-in-Amazon-VPC-across-availability-zones/ 

