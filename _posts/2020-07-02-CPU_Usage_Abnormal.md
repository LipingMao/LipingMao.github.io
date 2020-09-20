---
title: DevOps -- CPU Usage abnormal
---

### 问题：
线上一台服务器CPU User / Sys使用率忽然上升了15%，但是没有任何程序在使用这些CPU。


### 原因：


查看Promtheus中CPU的数据，发现问题时间点user/sys都涨了15%，但是Idle空闲97%,而mpstat和top的结果是，idle下降了，两者数据不一致：
```
[root@sh01gslb001 ~]# mpstat -P ALL 1
Linux 3.10.0-1062.9.1.el7.x86_64 (sh01gslb001.pano.video)   08/13/2020  _x86_64_    (4 CPU)
 
01:28:08 PM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
01:28:09 PM  all    7.88    0.00    9.41    0.22    0.00    0.00    0.00    0.00    0.00   82.49
01:28:09 PM    0    8.18    0.00    7.27    0.91    0.00    0.00    0.00    0.00    0.00   83.64
01:28:09 PM    1    8.94    0.00   13.01    0.00    0.00    0.00    0.00    0.00    0.00   78.05
01:28:09 PM    2    6.54    0.00    5.61    0.00    0.00    0.00    0.00    0.00    0.00   87.85
01:28:09 PM    3    8.47    0.00   11.02    0.00    0.00    0.00    0.00    0.00    0.00   80.51
```

最终定位是linux内核bug导致sys/user的cpu统计值不准确：
https://github.com/torvalds/linux/commit/9d7fb04276481c59610983362d8e023d262b58ca


对业务没有影响，重启服务器可以解决这个问题。
