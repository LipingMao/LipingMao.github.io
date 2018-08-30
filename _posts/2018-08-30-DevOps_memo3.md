---
title: ifconfig dropped packets
---

## Background

> ifconfig网卡时，会看到rx/tx处理包情况，dropped packets是常见情况的一种，这里简单总结了dropped packets的几种情况。


## What is the problem?

查看网卡ifconfig时，发现dropped packet增长。<br>

```
[root@apsj1mtc002 ~]# ifconfig
ens192: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.252.4.251  netmask 255.255.254.0  broadcast 10.252.5.255
        ether 00:50:56:b8:ff:c5  txqueuelen 1000  (Ethernet)
        RX packets 1583069923  bytes 316540800692 (294.8 GiB)
        RX errors 0  dropped 23234217  overruns 0  frame 0
        TX packets 1646497281  bytes 455040201729 (423.7 GiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```





## What is the root cause?

通常怀疑网卡rx队列来不及处理，可以通过ethtool -S $INTERFACE查看接口统计信息<br>
```
[root@apsj1mtc002 ~]# ethtool  -S ens192
NIC statistics:
     Tx Queue#: 0
       TSO pkts tx: 3777380
       TSO bytes tx: 15961950430
       ucast pkts tx: 318623436
       ucast bytes tx: 70944809850
       mcast pkts tx: 0
       mcast bytes tx: 0
       bcast pkts tx: 234
       bcast bytes tx: 9828
       pkts tx err: 0
       pkts tx discard: 0
       drv dropped tx total: 0
          too many frags: 0
          giant hdr: 0
          hdr err: 0
          tso: 0
       ring full: 0
       pkts linearized: 0
       hdr cloned: 0
       giant hdr: 0
...
Rx Queue#: 7
  LRO pkts rx: 270754
  LRO byte rx: 1692780469
  ucast pkts rx: 235905170
  ucast bytes rx: 42806582331
  mcast pkts rx: 30
  mcast bytes rx: 2580
  bcast pkts rx: 94761
  bcast bytes rx: 23845325
  pkts rx OOB: 0
  pkts rx err: 0
  drv dropped rx total: 0
     err: 0
     fcs: 0
  rx buf alloc fail: 0
tx timeout count: 0
```
观察发现接口没有因为rx队列原因丢包。<br>
之后可以观察softnet_stat是否因为软中断丢包。输出的第二列是丢包值。
```
[root@apsj1mtc002 ~]# cat /proc/net/softnet_stat
0ac7342a 00000000 00000099 00000000 00000000 00000000 00000000 00000000 00000000 00000000
0a41ec9a 00000000 0000009a 00000000 00000000 00000000 00000000 00000000 00000000 00000000
0db7eaea 00000000 0000007f 00000000 00000000 00000000 00000000 00000000 00000000 00000000
14561648 00000000 000000cc 00000000 00000000 00000000 00000000 00000000 00000000 00000000
0d68548e 00000000 0000009a 00000000 00000000 00000000 00000000 00000000 00000000 00000000
06862347 00000000 00000053 00000000 00000000 00000000 00000000 00000000 00000000 00000000
069f0856 00000000 0000007b 00000000 00000000 00000000 00000000 00000000 00000000 00000000
17860fd3 00000000 00000104 00000000 00000000 00000000 00000000 00000000 00000000 00000000
```
第二列都是0，可以看到没有因为软中断丢包。<br>
通过netstat -s命令查看高层协议是否丢包。
```
[root@apsj1mtc002 ~]# netstat -s
Ip:
    1695947913 total packets received
    0 forwarded
    144 with unknown protocol
    0 incoming packets discarded
    1695261595 incoming packets delivered
    1820421982 requests sent out
    26 dropped because of missing route
Icmp:
    1720889 ICMP messages received
    27653 input ICMP message failed.
    ICMP input histogram:
        destination unreachable: 1192
        timeout in transit: 27546
        echo requests: 1655508
        echo replies: 36451
        timestamp request: 48
        address mask request: 144
    1705634 ICMP messages sent
    0 ICMP messages failed
    ICMP output histogram:
        destination unreachable: 11935
        echo request: 38143
        echo replies: 1655508
        timestamp replies: 48
IcmpMsg:
        InType0: 36451
        InType3: 1192
        InType8: 1655508
        InType11: 27546
        InType13: 48
        InType17: 144
        OutType0: 1655508
        OutType3: 11935
        OutType8: 38143
        OutType14: 48
Tcp:
    17228801 active connections openings
    41854512 passive connection openings
    390941 failed connection attempts
    737192 connection resets received
    32 connections established
    1680312797 segments received
    1925016373 segments send out
    457161 segments retransmited
    731 bad segments received.
    3356677 resets sent
    InCsumErrors: 613
Udp:
    5558288 packets received
    39805 packets to unknown port received.
    0 packet receive errors
    12408776 packets sent
    0 receive buffer errors
    0 send buffer errors
UdpLite:
TcpExt:
    24780 invalid SYN cookies received
    6568 resets received for embryonic SYN_RECV sockets
    564 ICMP packets dropped because they were out-of-window
    49845763 TCP sockets finished time wait in fast timer
    2 packets rejects in established connections because of timestamp
    40878227 delayed acks sent
    10269 delayed acks further delayed because of locked socket
    Quick ack mode was activated 14588 times
    1 SYNs to LISTEN sockets dropped
    29504492 packets directly queued to recvmsg prequeue.
    199638574 bytes directly in process context from backlog
    129321359 bytes directly received in process context from prequeue
    648531311 packet headers predicted
    422779 packets header predicted and directly queued to user
    191880912 acknowledgments not containing data payload received
    194427246 predicted acknowledgments
    799 times recovered from packet loss by selective acknowledgements
    Detected reordering 5 times using FACK
    Detected reordering 12 times using SACK
    Detected reordering 109 times using time stamp
    66 congestion windows fully recovered without slow start
    116 congestion windows partially recovered using Hoe heuristic
    232 congestion windows recovered without slow start by DSACK
    778 congestion windows recovered without slow start after partial ack
    TCPLostRetransmit: 27
    209 timeouts after SACK recovery
    68 timeouts in loss state
    852 fast retransmits
    238 forward retransmits
    194 retransmits in slow start
    30207 other TCP timeouts
    TCPLossProbes: 439407
    TCPLossProbeRecovery: 330741
    72 SACK retransmits failed
    14535 DSACKs sent for old packets
    3 DSACKs sent for out of order packets
    332706 DSACKs received
    18 DSACKs for out of order packets received
    2129053 connections reset due to unexpected data
    466562 connections reset due to early user close
    16267 connections aborted due to timeout
    TCPSACKDiscard: 1
    TCPDSACKIgnoredOld: 5
    TCPDSACKIgnoredNoUndo: 323608
    TCPSpuriousRTOs: 163
    TCPSackShifted: 341
    TCPSackMerged: 340
    TCPSackShiftFallback: 7465
    TCPDeferAcceptDrop: 139214
    IPReversePathFilter: 349266
    TCPRetransFail: 3
    TCPRcvCoalesce: 93767091
    TCPOFOQueue: 2639
    TCPOFOMerge: 10
    TCPChallengeACK: 129
    TCPSYNChallenge: 118
    TCPSpuriousRtxHostQueues: 125
    TCPAutoCorking: 54719518
    TCPFromZeroWindowAdv: 63
    TCPToZeroWindowAdv: 63
    TCPWantZeroWindowAdv: 332
    TCPSynRetrans: 120454
    TCPOrigDataSent: 1055190015
    TCPHystartTrainDetect: 60140
    TCPHystartTrainCwnd: 1192596
    TCPHystartDelayDetect: 3
    TCPHystartDelayCwnd: 1054
    TCPACKSkippedSynRecv: 443
    TCPACKSkippedChallenge: 5
IpExt:
    InNoRoutes: 6
    InMcastPkts: 533689
    InBcastPkts: 7096115
    InOctets: 305482739716
    OutOctets: 512822988829
    InMcastOctets: 19212804
    InBcastOctets: 643051144
    InNoECTPkts: 1695947675
    InECT0Pkts: 238
```

在内核升级到2.6.37之后以下情况也会算作interface drop包。[参见blog](http://www.361way.com/ifconfig-dropped-rx-packets/5722.html)
```
Softnet backlog full  -- (Measured from /proc/net/softnet_stat)
Bad / Unintended VLAN tags
Unknown / Unregistered protocols
IPv6 frames when the server is not configured for IPv6
```
可以通过观察tcpdump过程中，接口是否丢包来判断是否是上述情况。进过测试发现在tcpdump过程中，没有任何丢包。基本断定是这4种情况。经过tcpdump发现存在Unkown protocol，因此产生丢包。
```
[root@apsj1mtc002 ~]# tcpdump -i ens192 -nnn -eee -vvv | grep "pid Unknown"
tcpdump: listening on ens192, link-type EN10MB (Ethernet), capture size 65535 bytes
14:52:44.189652 00:50:56:8f:e4:a5 > ff:ff:ff:ff:ff:ff, 802.3, length 486: LLC, dsap SNAP (0xaa) Individual, ssap SNAP (0xaa) Command, ctrl 0x03: oui Cisco (0x00000c), pid Unknown (0x0dec): Unnumbered, ui, Flags [Command], length 472
14:52:45.189761 00:50:56:8f:e4:a5 > ff:ff:ff:ff:ff:ff, 802.3, length 486: LLC, dsap SNAP (0xaa) Individual, ssap SNAP (0xaa) Command, ctrl 0x03: oui Cisco (0x00000c), pid Unknown (0x0dec): Unnumbered, ui, Flags [Command], length 472
14:52:46.189911 00:50:56:8f:e4:a5 > ff:ff:ff:ff:ff:ff, 802.3, length 486: LLC, dsap SNAP (0xaa) Individual, ssap SNAP (0xaa) Command, ctrl 0x03: oui Cisco (0x00000c), pid Unknown (0x0dec): Unnumbered, ui, Flags [Command], length 472
```



## What is the solution?

This is action of dropped stats (kernel >= 2.6.37)  by design...
