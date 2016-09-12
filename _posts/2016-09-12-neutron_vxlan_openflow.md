---
title: Neutron Memo - vxlan basic openflow rules
---
## Backgroup

> The blog will introduce how vxlan works in neutron basically.

## Flowtable on br-tun

Here is a openflow sample for br-tun on compute node.

```
# ovs-ofctl dump-flows br-tun
NXST_FLOW reply (xid=0x4):
 cookie=0x899a587eff12b66a, duration=179004.777s, table=0, n_packets=448, n_bytes=42502, idle_age=0, hard_age=65534, priority=1,in_port=1 actions=resubmit(,2)
 cookie=0x899a587eff12b66a, duration=179004.390s, table=0, n_packets=14, n_bytes=2326, idle_age=3138, hard_age=65534, priority=1,in_port=4 actions=resubmit(,4)
 cookie=0x899a587eff12b66a, duration=179004.371s, table=0, n_packets=89752, n_bytes=4859059, idle_age=0, hard_age=65534, priority=1,in_port=5 actions=resubmit(,4)
 cookie=0x899a587eff12b66a, duration=179004.352s, table=0, n_packets=31, n_bytes=5325, idle_age=2614, hard_age=65534, priority=1,in_port=6 actions=resubmit(,4)
 cookie=0x899a587eff12b66a, duration=179004.332s, table=0, n_packets=89480, n_bytes=4835402, idle_age=0, hard_age=65534, priority=1,in_port=7 actions=resubmit(,4)
 cookie=0x899a587eff12b66a, duration=179004.312s, table=0, n_packets=0, n_bytes=0, idle_age=65534, hard_age=65534, priority=1,in_port=2 actions=resubmit(,4)
 cookie=0x899a587eff12b66a, duration=179004.293s, table=0, n_packets=0, n_bytes=0, idle_age=65534, hard_age=65534, priority=1,in_port=3 actions=resubmit(,4)
 cookie=0x899a587eff12b66a, duration=179004.776s, table=0, n_packets=0, n_bytes=0, idle_age=65534, hard_age=65534, priority=0 actions=drop
 cookie=0x899a587eff12b66a, duration=179004.776s, table=2, n_packets=388, n_bytes=38126, idle_age=0, hard_age=65534, priority=0,dl_dst=00:00:00:00:00:00/01:00:00:00:00:00 actions=resubmit(,20)
 cookie=0x899a587eff12b66a, duration=179004.776s, table=2, n_packets=60, n_bytes=4376, idle_age=5, hard_age=65534, priority=0,dl_dst=01:00:00:00:00:00/01:00:00:00:00:00 actions=resubmit(,22)
 cookie=0x899a587eff12b66a, duration=179004.775s, table=3, n_packets=0, n_bytes=0, idle_age=65534, hard_age=65534, priority=0 actions=drop
 cookie=0x899a587eff12b66a, duration=179003.910s, table=4, n_packets=249, n_bytes=26302, idle_age=0, hard_age=65534, priority=1,tun_id=0xeaae actions=mod_vlan_vid:2,resubmit(,10)
 cookie=0x899a587eff12b66a, duration=179003.884s, table=4, n_packets=124, n_bytes=14994, idle_age=2745, hard_age=65534, priority=1,tun_id=0xeac2 actions=mod_vlan_vid:1,resubmit(,10)
 cookie=0x899a587eff12b66a, duration=179004.775s, table=4, n_packets=178904, n_bytes=9660816, idle_age=0, hard_age=65534, priority=0 actions=drop
 cookie=0x899a587eff12b66a, duration=179004.775s, table=6, n_packets=0, n_bytes=0, idle_age=65534, hard_age=65534, priority=0 actions=drop
 cookie=0x899a587eff12b66a, duration=179004.774s, table=10, n_packets=373, n_bytes=41296, idle_age=0, hard_age=65534, priority=1 actions=learn(table=20,hard_timeout=300,priority=1,cookie=0x899a587eff12b66a,NXM_OF_VLAN_TCI[0..11],NXM_OF_ETH_DST[]=NXM_OF_ETH_SRC[],load:0->NXM_OF_VLAN_TCI[],load:NXM_NX_TUN_ID[]->NXM_NX_TUN_ID[],output:NXM_OF_IN_PORT[]),output:1
 cookie=0x899a587eff12b66a, duration=5.359s, table=20, n_packets=7, n_bytes=630, hard_timeout=300, idle_age=0, hard_age=0, priority=1,vlan_tci=0x0002/0x0fff,dl_dst=fa:16:3e:74:04:c7 actions=load:0->NXM_OF_VLAN_TCI[],load:0xeaae->NXM_NX_TUN_ID[],output:5
 cookie=0x899a587eff12b66a, duration=179004.774s, table=20, n_packets=1, n_bytes=98, idle_age=2750, hard_age=65534, priority=0 actions=resubmit(,22)
 cookie=0x899a587eff12b66a, duration=519122.852s, table=22, n_packets=109, n_bytes=10226, idle_age=5, hard_age=65534, dl_vlan=2 actions=strip_vlan,set_tunnel:0xeaae,output:3,output:4,output:5,output:6,output:7,output:2
 cookie=0x899a587eff12b66a, duration=436067.673s, table=22, n_packets=51, n_bytes=3862, idle_age=2750, hard_age=65534, dl_vlan=1 actions=strip_vlan,set_tunnel:0xeac2,output:3,output:4,output:5,output:6,output:7,output:2
 cookie=0x899a587eff12b66a, duration=179004.761s, table=22, n_packets=5, n_bytes=450, idle_age=65534, hard_age=65534, priority=0 actions=drop
``` 

Life is not easy, Let's go through these flow one by one.

```
Table0 |--from internal---> Table2 |--unicast  --> Table20
       |                           |--broadcast--> Table22
       |
       |--from outside ---> Table4 --tun_id to local_vlan--> Table10(Mac learning) 
```

Table0: 
when the traffic comes from br-int, they resubmit the traffic to Table2.
when the traffic comes from vxlan, they resubmit the traffic to Table4.
In other case, they drop the packet.

```
 cookie=0x899a587eff12b66a, duration=179004.777s, table=0, n_packets=448, n_bytes=42502, idle_age=0, hard_age=65534, priority=1,in_port=1 actions=resubmit(,2)
 cookie=0x899a587eff12b66a, duration=179004.390s, table=0, n_packets=14, n_bytes=2326, idle_age=3138, hard_age=65534, priority=1,in_port=4 actions=resubmit(,4)
 cookie=0x899a587eff12b66a, duration=179004.371s, table=0, n_packets=89752, n_bytes=4859059, idle_age=0, hard_age=65534, priority=1,in_port=5 actions=resubmit(,4)
 cookie=0x899a587eff12b66a, duration=179004.352s, table=0, n_packets=31, n_bytes=5325, idle_age=2614, hard_age=65534, priority=1,in_port=6 actions=resubmit(,4)
 cookie=0x899a587eff12b66a, duration=179004.332s, table=0, n_packets=89480, n_bytes=4835402, idle_age=0, hard_age=65534, priority=1,in_port=7 actions=resubmit(,4)
 cookie=0x899a587eff12b66a, duration=179004.312s, table=0, n_packets=0, n_bytes=0, idle_age=65534, hard_age=65534, priority=1,in_port=2 actions=resubmit(,4)
 cookie=0x899a587eff12b66a, duration=179004.293s, table=0, n_packets=0, n_bytes=0, idle_age=65534, hard_age=65534, priority=1,in_port=3 actions=resubmit(,4)
 cookie=0x899a587eff12b66a, duration=179004.776s, table=0, n_packets=0, n_bytes=0, idle_age=65534, hard_age=65534, priority=0 actions=drop
```

Table2:
If packet is unicast, send to Table20, if packet is broadcast, send to Table22:

```
 cookie=0x899a587eff12b66a, duration=179004.776s, table=2, n_packets=388, n_bytes=38126, idle_age=0, hard_age=65534, priority=0,dl_dst=00:00:00:00:00:00/01:00:00:00:00:00 actions=resubmit(,20)
 cookie=0x899a587eff12b66a, duration=179004.776s, table=2, n_packets=60, n_bytes=4376, idle_age=5, hard_age=65534, priority=0,dl_dst=01:00:00:00:00:00/01:00:00:00:00:00 actions=resubmit(,22)
```

Table4:
Modify the tun_id to local vlan id, then send to Table10, if tun_id not exist, drop the packet.

```
 cookie=0x899a587eff12b66a, duration=179003.910s, table=4, n_packets=249, n_bytes=26302, idle_age=0, hard_age=65534, priority=1,tun_id=0xeaae actions=mod_vlan_vid:2,resubmit(,10)
 cookie=0x899a587eff12b66a, duration=179003.884s, table=4, n_packets=124, n_bytes=14994, idle_age=2745, hard_age=65534, priority=1,tun_id=0xeac2 actions=mod_vlan_vid:1,resubmit(,10)
 cookie=0x899a587eff12b66a, duration=179004.775s, table=4, n_packets=178904, n_bytes=9660816, idle_age=0, hard_age=65534, priority=0 actions=drop
```

Table10:
Learn the TUN_ID, vxlan of_port, tun_id to create flow in Table20.

```
 cookie=0x899a587eff12b66a, duration=179004.774s, table=10, n_packets=373, n_bytes=41296, idle_age=0, hard_age=65534, priority=1 actions=learn(table=20,hard_timeout=300,priority=1,cookie=0x899a587eff12b66a,NXM_OF_VLAN_TCI[0..11],NXM_OF_ETH_DST[]=NXM_OF_ETH_SRC[],load:0->NXM_OF_VLAN_TCI[],load:NXM_NX_TUN_ID[]->NXM_NX_TUN_ID[],output:NXM_OF_IN_PORT[]),output:1
```

Table20:
The learned flow will exist 300 sec, if there is learned flow, send to Table22.

```
 cookie=0x899a587eff12b66a, duration=5.359s, table=20, n_packets=7, n_bytes=630, hard_timeout=300, idle_age=0, hard_age=0, priority=1,vlan_tci=0x0002/0x0fff,dl_dst=fa:16:3e:74:04:c7 actions=load:0->NXM_OF_VLAN_TCI[],load:0xeaae->NXM_NX_TUN_ID[],output:5
 cookie=0x899a587eff12b66a, duration=179004.774s, table=20, n_packets=1, n_bytes=98, idle_age=2750, hard_age=65534, priority=0 actions=resubmit(,22)
```

Table22:

Change local vlan to tun_id, and broadcast the traffic to all the tun, if the local vlan not exist, just drop the packet.

```
 cookie=0x899a587eff12b66a, duration=519122.852s, table=22, n_packets=109, n_bytes=10226, idle_age=5, hard_age=65534, dl_vlan=2 actions=strip_vlan,set_tunnel:0xeaae,output:3,output:4,output:5,output:6,output:7,output:2
 cookie=0x899a587eff12b66a, duration=436067.673s, table=22, n_packets=51, n_bytes=3862, idle_age=2750, hard_age=65534, dl_vlan=1 actions=strip_vlan,set_tunnel:0xeac2,output:3,output:4,output:5,output:6,output:7,output:2
 cookie=0x899a587eff12b66a, duration=179004.761s, table=22, n_packets=5, n_bytes=450, idle_age=65534, hard_age=65534, priority=0 actions=drop
```
