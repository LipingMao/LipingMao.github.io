---
title: Journey to fight with Openstack Network
---
## Backgroup

> Openstack is a widely used solution for private cloud. We are using Openstack
as the Infra Layer in Cisco Webex. Since Webex Service is realtime media
Service, it will has large network traffic especially udp traffic. This blog
is the journey we fight with Openstack Network ;-)


## Overall

> Network Stack in linux is very complex, network in Openstack is more complex.
Linux Network Stack is very import part in most of Opestack Solution. We focus
on the test/tun/monitor network datapath of openstack in this blog.


## Environment

>In Webex, we have two main Openstack Version used in production
(Juno and Mitaka). We use vlan segment as default tenant network segment in
Juno and use vxlan segment in Mitaka. Mitaka are upgrading to Newton recently,
so all my test is based on Newton version.

Here is detail info:
```
Hardware: Cisco UCS C220M3
OS: RHEL7.3
Openvswitch: 2.5.0
Segment: vxlan for default tenant network, vlan for provider network
```
