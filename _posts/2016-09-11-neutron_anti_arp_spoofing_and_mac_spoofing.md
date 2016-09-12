---
title: Neutron Memo(2) -- arp-spoofing / mac-spoofing
---
## Backgroup

> Neutron has security for arp spoofing and mac spoofing by default. The blog will introduce what's arp spoofing and mac spoofing, how neutron do it in openvswitch.


## What's ARP Spoofing and MAC Spoofing?

After VM boot in Openstack, the VM will get mac and ip from neutron. Technically, End User can change the IP and MAC in the VM, by this way, user can attack other VMs. 


### ARP Spoofing

VMs will use ARP to get MAC address, Attacker can send out ARP response with his IP to fresh other VM's the ARP Cache. So Neutron will only allow VM send out ARP response with its IP by default.


### MAC Spoofing

VMs technically can send out packet with other MACs which is not allocated by Neutron. So Neutron will only allow VM send out MAC with neutron allocated.


Note:
Neutron use iptables to anti-ip spoofing(only allowed send out IP packet with the ip address which allocated by neutron), this blog will not cover this part. 


## How it works in neutron openvswitch?

Here is openflow rule sample of br-int on the compute node.

```
# ovs-ofctl dump-flows br-int
NXST_FLOW reply (xid=0x4):
 cookie=0xa6623d55bf2e8cea, duration=151319.882s, table=0, n_packets=0, n_bytes=0, idle_age=65534, hard_age=65534, priority=10,icmp6,in_port=28,icmp_type=136 actions=resubmit(,24)
 cookie=0xa6623d55bf2e8cea, duration=151319.872s, table=0, n_packets=6, n_bytes=252, idle_age=23109, hard_age=65534, priority=10,arp,in_port=28 actions=resubmit(,24)
 cookie=0xa6623d55bf2e8cea, duration=153193.302s, table=0, n_packets=114905, n_bytes=7847569, idle_age=0, hard_age=65534, priority=2,in_port=1 actions=drop
 cookie=0xa6623d55bf2e8cea, duration=151319.893s, table=0, n_packets=3, n_bytes=996, idle_age=23114, hard_age=65534, priority=9,in_port=28 actions=resubmit(,25)
 cookie=0xa6623d55bf2e8cea, duration=153193.398s, table=0, n_packets=354, n_bytes=40338, idle_age=21443, hard_age=65534, priority=0 actions=NORMAL
 cookie=0xa6623d55bf2e8cea, duration=153193.392s, table=23, n_packets=0, n_bytes=0, idle_age=65534, hard_age=65534, priority=0 actions=drop
 cookie=0xa6623d55bf2e8cea, duration=151319.887s, table=24, n_packets=0, n_bytes=0, idle_age=65534, hard_age=65534, priority=2,icmp6,in_port=28,icmp_type=136,nd_target=fe80::f816:3eff:fefd:85f7 actions=NORMAL
 cookie=0xa6623d55bf2e8cea, duration=151319.877s, table=24, n_packets=6, n_bytes=252, idle_age=23109, hard_age=65534, priority=2,arp,in_port=28,arp_spa=200.200.200.110 actions=resubmit(,25)
 cookie=0xa6623d55bf2e8cea, duration=153193.387s, table=24, n_packets=0, n_bytes=0, idle_age=65534, hard_age=65534, priority=0 actions=drop
 cookie=0xa6623d55bf2e8cea, duration=151319.903s, table=25, n_packets=9, n_bytes=1248, idle_age=23109, hard_age=65534, priority=2,in_port=28,dl_src=fa:16:3e:fd:85:f7 actions=NORMAL
```

After the packet send out from the VM, it will go into br-int. and go through openflow tables from table 0 to table 255. Neutron use table 0, table 24, table 25 in this case. Let's focus on these tables first.

### Rules in table 0(LOCAL_SWITCHING)

Openflow will lookup flow table from high priority to low priority. 

priority 10:
    if the packet is from VM and ARP(ICMP4) or ICMP6 type 136(IPV6), then go to table 24(ARP_SPOOF_TABLE)

priority 9:
    if the packet is from VM, then go to table 25(MAC_SPOOF_TABLE)

priority 2:
    if the packet is from other Br(br-ex/br-eth0...), then drop it

priority 0:
     Normal action.

here is the flows:

```
 cookie=0xa6623d55bf2e8cea, duration=151319.882s, table=0, n_packets=0, n_bytes=0, idle_age=65534, hard_age=65534, priority=10,icmp6,in_port=28,icmp_type=136 actions=resubmit(,24)
 cookie=0xa6623d55bf2e8cea, duration=151319.872s, table=0, n_packets=6, n_bytes=252, idle_age=23109, hard_age=65534, priority=10,arp,in_port=28 actions=resubmit(,24)
 cookie=0xa6623d55bf2e8cea, duration=153193.302s, table=0, n_packets=114905, n_bytes=7847569, idle_age=0, hard_age=65534, priority=2,in_port=1 actions=drop
 cookie=0xa6623d55bf2e8cea, duration=151319.893s, table=0, n_packets=3, n_bytes=996, idle_age=23114, hard_age=65534, priority=9,in_port=28 actions=resubmit(,25)
 cookie=0xa6623d55bf2e8cea, duration=153193.398s, table=0, n_packets=354, n_bytes=40338, idle_age=21443, hard_age=65534, priority=0 actions=NORMAL
```

### Rules in table 24(ARP_SPOOF_TABLE)

Only resubmit to table 25 when the ip is allocated by neutron, drop other packets:

```
 cookie=0xa6623d55bf2e8cea, duration=151319.887s, table=24, n_packets=0, n_bytes=0, idle_age=65534, hard_age=65534, priority=2,icmp6,in_port=28,icmp_type=136,nd_target=fe80::f816:3eff:fefd:85f7 actions=NORMAL
 cookie=0xa6623d55bf2e8cea, duration=151319.877s, table=24, n_packets=6, n_bytes=252, idle_age=23109, hard_age=65534, priority=2,arp,in_port=28,arp_spa=200.200.200.110 actions=resubmit(,25)
 cookie=0xa6623d55bf2e8cea, duration=153193.387s, table=24, n_packets=0, n_bytes=0, idle_age=65534, hard_age=65534, priority=0 actions=drop
```

### Rules in table 25(MAC_SPOOF_TABLE)

Do normal switch only if the MAC address is allocated in neutron.

```
 cookie=0xa6623d55bf2e8cea, duration=151319.903s, table=25, n_packets=9, n_bytes=1248, idle_age=23109, hard_age=65534, priority=2,in_port=28,dl_src=fa:16:3e:fd:85:f7 actions=NORMAL
```

## Allowed address pairs feature impact

if you add allowed address pairs with MAC, the mac will be allowed here too. Here is a sample.

```
# Add allowed address pair
neutron port-update --allowed-address-pair ip_address=200.200.200.110,mac_address=fa:16:3e:fd:85:f8 0ff8b5b0-14b0-4116-a195-3e023e921496
Updated port: 0ff8b5b0-14b0-4116-a195-3e023e921496
```

You can find iptables rules updated in iptables anti ip spoofing chain.

```
Chain neutron-openvswi-s0ff8b5b0-1 (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 RETURN     all  --  *      *       200.200.200.110      0.0.0.0/0            MAC FA:16:3E:FD:85:F8 /* Allow traffic from defined IP/MAC pairs. */
    3   954 RETURN     all  --  *      *       200.200.200.110      0.0.0.0/0            MAC FA:16:3E:FD:85:F7 /* Allow traffic from defined IP/MAC pairs. */
    0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* Drop traffic without an IP/MAC allow rule. */
```

The flow table rules for anti-arp and anti-mac address are also updated.

```
# ovs-ofctl dump-flows br-int
...
 cookie=0xa6623d55bf2e8cea, duration=13.756s, table=24, n_packets=0, n_bytes=0, idle_age=13, priority=2,icmp6,in_port=28,icmp_type=136,nd_target=fe80::f816:3eff:fefd:85f8 actions=NORMAL
 cookie=0xa6623d55bf2e8cea, duration=13.750s, table=24, n_packets=0, n_bytes=0, idle_age=13, priority=2,icmp6,in_port=28,icmp_type=136,nd_target=fe80::f816:3eff:fefd:85f7 actions=NORMAL
 cookie=0xa6623d55bf2e8cea, duration=13.740s, table=24, n_packets=0, n_bytes=0, idle_age=13, priority=2,arp,in_port=28,arp_spa=200.200.200.110 actions=resubmit(,25)
...
 cookie=0xa6623d55bf2e8cea, duration=13.779s, table=25, n_packets=0, n_bytes=0, idle_age=13, priority=2,in_port=28,dl_src=fa:16:3e:fd:85:f8 actions=NORMAL
 cookie=0xa6623d55bf2e8cea, duration=13.774s, table=25, n_packets=0, n_bytes=0, idle_age=13, priority=2,in_port=28,dl_src=fa:16:3e:fd:85:f7 actions=NORMAL
```
