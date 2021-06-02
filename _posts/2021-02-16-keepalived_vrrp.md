---
title: DevOps -- keepalived vrrp
---

> 在云环境发现Keepalived控制的Redis发生双主状态，定位时，发现是因为Keepalived在很短时间内进行了master/slave的切换。顺便看了VRRP协议相关内容。简单总结一下VRRP。

VRRP目前是V3版本，协议如下：

```
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |Version| Type  | Virtual Rtr ID|   Priority    | Count IP Addrs|
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |   Auth Type   |   Adver Int   |          Checksum             |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                         IPvX Address (1)                      |
   |                            .                                  |
   |                            .                                  |
   |                            .                                  |
   |                         IPvX Address (n)                      |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

```

可以看到协议中会携带Virtual Rtr ID / Priority / Adver Int这几个字段是和选主有关。其中Adver Int代表发送VRRP心跳的频率。默认情况下是1秒。而在另外一端，如果超过一定时间没有收到心跳包，则会发生状态切换。

超时时间是:
```
   Skew_Time                   Time to skew Master_Down_Interval in
                               centiseconds.  Calculated as

                   (((256 - priority) * Master_Adver_Interval) / 256)

   Master_Down_Interval        Time interval for Backup to declare
                               Master down (centiseconds).
                               Calculated as

                               (3 * Master_Adver_Interval) + Skew_time
```

可以看到默认情况下，超时时间是3-4s之间，Skew_time是与priority反比的Adver Int。
