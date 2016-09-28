---
title: Neutron Memo(4) -- qos
---
## Backgroup

> The blog will have a basic introduce the neutron qos how to in Mitaka.

## How to enable qos in neutron

Pleasae refer [Network Guide](http://docs.openstack.org/mitaka/networking-guide/config-qos.html)


## How to use qos feature

You need to create qos policy and rule first:
```
[root@ci91hf3ctr001 ~]# neutron qos-policy-create limao_test
Created a new policy:
+-------------+--------------------------------------+
| Field       | Value                                |
+-------------+--------------------------------------+
| description |                                      |
| id          | 43cf925c-6152-4d83-a624-f8093aae71dc |
| name        | limao_test                           |
| rules       |                                      |
| shared      | False                                |
| tenant_id   | fa7c66615e8948039ec06f2bf7c13be3     |
+-------------+--------------------------------------+
[root@ci91hf3ctr001 ~]# neutron qos-bandwidth-limit-rule-create --max-kbps 1024  --max-burst-kbps 1024 limao_test
Created a new bandwidth_limit_rule:
+----------------+--------------------------------------+
| Field          | Value                                |
+----------------+--------------------------------------+
| id             | f9a69e2f-7d0b-43a1-b8e8-a8293c36809d |
| max_burst_kbps | 1024                                 |
| max_kbps       | 1024                                 |
+----------------+--------------------------------------+
```

You can either bind the qos policy to network level or bind it to port level:
```
[root@ci91hf3ctr001 ~]# neutron net-update --qos-policy limao_test $NET_UUID


[root@ci91hf3ctr001 ~]# neutron port-update --qos-policy limao_test $PORT_UUID
```

if you are using ovs, it will set up to ingress_policing_burst and ingress_policing_rate:
```
[root@ci91hf3cmp002 ~]# ovs-vsctl list interface
_uuid               : a0d005b6-7c94-4e8f-af6b-531c5fc022dc
admin_state         : up
...
external_ids        : {attached-mac="fa:16:3e:00:aa:29", iface-id="742f21ea-09dd-4777-9d0a-77ca3ffe216c", iface-status=active, vm-uuid="83d4cb38-11f9-49d4-bc22-8b78de234baf"}
ifindex             : 171
ingress_policing_burst: 1024
ingress_policing_rate: 1024
...
statistics          : {collisions=0, rx_bytes=701350407, rx_crc_err=0, rx_dropped=0, rx_errors=0, rx_frame_err=0, rx_over_err=0, rx_packets=9758885, tx_bytes=37686030361, tx_dropped=0, tx_errors=0, tx_packets=19241057}
status              : {driver_name=veth, driver_version="1.0", firmware_version=""}
type                : ""
```

Note, This qos means the egress traffic from the virtual machine, for ingress traffic, there is no limit.
For example, if the qos of a vm is 10Mbps, the traffic goes out from VMs will be 10Mbps at most. But for ingress traffic, there is no limit.
