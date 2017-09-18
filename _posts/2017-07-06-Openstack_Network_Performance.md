---
title: Tun container datapath performance with kuryr-libnetowrk upon openstack
---
## Backgroup

> Openstack is a widely used solution for private cloud. We are using Openstack
as the Infra Layer in Cisco Webex. Since Webex Service is realtime media
Service, it will has large network traffic especially udp traffic. This blog
is the journey we fight with Openstack Network ;-)


## Overall

> Network Stack in linux is very complex, network in Openstack is more complex.
Linux Network Stack is very important part in most of Opestack Solution. We focus
on the test/tun/monitor network datapath of openstack in this blog.


## Environment

>In Webex, we have two main Openstack Version used in production
(Juno and Mitaka). We use vlan segment as default tenant network segment in
Juno and use vxlan segment in Mitaka. Mitaka are upgrading to Newton recently,
so all my test is based on Newton version. I use iperf3 as the network
performance benchmark tool.


Here is Environment info:
```
Hardware: Cisco UCS C220M3
OS: RHEL7.3
Openvswitch: 2.5.0
Segment: vxlan for default tenant network, vlan for provider network
```

## Neutron Network and Linux Network background

We will not cover neutron network and linux network detail in this blog, here is some good blog we can refer:
https://blog.packagecloud.io/eng/2016/06/22/monitoring-tuning-linux-networking-stack-receiving-data/
https://blog.packagecloud.io/eng/2017/02/06/monitoring-tuning-linux-networking-stack-sending-data/
https://docs.openstack.org/newton/networking-guide/



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

### Physical Interface Performance Before tun

> Let's have a quick look how the performance looks like from two physical interface.

I use iperf3 as performance test tool
```
On Client Side:
  iperf3 --client 10.225.18.15 -u -w 4M --bandwidth 200M  --length 64 --port 5201 -t 1000
On Server Side:
  iperf3 -s -p 5201
```

The throughput can reach to about 200-250kpps before drop packets, after it reach to 250-300kpps, physical interface start to drop packets.

You can check

By default, UCS VNIC driver only have 4 rx queue with queue 512, 1 tx queue with length 256.


1. Tunning RX queue length from 512 to 4096(which is the max length of rx queue)
after queue length increase to 4096, the throughput of one rx queue reach to around 300-350kpps , and start to drop around 350-400kpps.

+ IRQ CPU picture
+ SoftIRQ
+ CPU si usage

2.
After tun sysctl -w net.core.netdev_budget=1000
350kpps -400kpps not drop packet
400kpps-450kpps start drop packet

+ picture
+ softnet status


3. Tunning RX number
RSS will schedule to different CPU if source
RSS will distribute different flow to different CPU cores.  
For UDP, cisco enic driver will distribute the flow on different cpu core by hash source/dest ipv4, I send UDP on two different host. It can almost reach to 700kpps-750kpps, we can see the interrupt distribute to two differenct cpu cores, softirq is on two different CPU.

Two CPU:
+ IRQ CPU picture
+ SoftIRQ
+ CPU si usage

Three CPU:
how it looks like


How to update RX/TX Queue number in UCS:
Update rx/tx queue number.

Note that:
Default Interrupt Count is 8 in vNIC, that means the total Interrupt it can allocate for nic, each rx, tx queue need one Interrupt, and by default, we have $DEVICE-err and $DEVICE-notify interrupt. So if we update rx/tx queue , we also need to update Interrupt Count here, in this case, we use 8 rx queue and 8 tx queue , and Interrupt Count 18 (it need at least 8 +8 +2 here). Completion Queue Count also need to update to rx + tx queue count.


Interrupt Count = rx queue + tx queue +2
Completion Queue Count = rx queue + tx queue

+ picture

How RSS works ?

+ UDP RSS
+ TCP RSS



## Virtual Switch Layer

### OVS forward

OVS internal interface to internal interface, pps can reach to 3.3Mbps with 32cpu without drop packet.
The problem is because we also run iperf server/client on this host, cpu is used up by kernel and iperf under this load.
It can't grow more by run more iperf since all cpu is used up.

+ Picture


### VM to VM

+ Tap device length


+ multi-queue


+ VM to VM vxlan Performance

+ VM to VM vlan Performance

+ floating ip with vlan Performance

+ floating ip with vxlan Performance

Conntrack max in namespace
tap device length
net buget





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
provider network instead of
```


## Container Layer

```
TODO: original overlay / bridge
kuryr MACVLAN/IPVLAN?
kuryr paththrough / sriov
baremetal server + kuryr performance
different ip address matters!!!
```


## Key take aways TIPS

```
TODO: summary key take aways
```
