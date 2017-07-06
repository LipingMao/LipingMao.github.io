---
title: Journey to fight with Openstack Network(Draft and WIP)
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

## Openstack Network Overall

Here is a overall introduce of Openstack Network:
```
TODO: add openstack network overall introduce
Network Node detail
Compute Node detail
```

## Linux Network Stack

It is very important to understand how linux receive packet and send packet.
Luckly we have this blog:
TODO: add two blog link, and update the words here.


## Physical Layer Performance

```
 TODO: this part will introduce the physical layer performance test/tun/monitor
physical lay connect arch
talk about : RX Queue len/number , TX queue len/number, RSS, offload.
Impact and test result
Performance before update and after update.
Tips to monitor: physical interface drop packet number
per cpu usage, especially care about iq
/proc/net/soft...
/proc/interrupt to check if RSS works well
```


## Virtual Switch Layer

```
TODO: this part will introduce the virtual switch layer, ovs, tap
talk about: vxlan and vlan of ovs Performance
tap device length
multi-queue
net buget
netfilter conntrack max
tips to monitor:
tap device drop packet and so on
conntrack count and max
net buget , cpu usage
```


## Inside VM

```
TODO: virtio driver? udp/tcp params need to update
udp packet length
/proc/net/snmp in kernel layer
cpu iq / cpu steal
```


## Network Node

```
TODO: vrouter performance and limit
vxlan / vlan performance inside vrouter
performance impact of netfilter
vlanXXX monitor for vxlan endpoint
limit vrouter number on one network node in L3 schedule and Metadata
```


## Key take aways TIPS

```
TODO: summary key take aways
```
