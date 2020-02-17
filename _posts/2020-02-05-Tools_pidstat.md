---
title: DevOps -- pidstat
---

> pidstat可以定位进程/线程级别资源使用情况。

### Install

pidstat不是系统默认自带的程序，需要安装sysstat组件。

### Usage Sample

只看一个命令：pidstat -C COMMAND
```
[root@officeci001 ~]# pidstat -C cadvisor 1
Linux 3.10.0-957.21.3.el7.x86_64 (officeci001.pano.video) 	02/17/2020 	_x86_64_	(2 CPU)

02:43:48 PM   UID       PID    %usr %system  %guest    %CPU   CPU  Command

02:43:49 PM   UID       PID    %usr %system  %guest    %CPU   CPU  Command
02:43:50 PM     0     32673    0.00    1.00    0.00    1.00     0  cadvisor

02:43:50 PM   UID       PID    %usr %system  %guest    %CPU   CPU  Command
02:43:51 PM     0     32673    1.00    0.00    0.00    1.00     0  cadvisor

02:43:51 PM   UID       PID    %usr %system  %guest    %CPU   CPU  Command
02:43:52 PM     0     32673    0.00    1.00    0.00    1.00     0  cadvisor

02:43:52 PM   UID       PID    %usr %system  %guest    %CPU   CPU  Command
02:43:53 PM     0     32673    2.00    0.00    0.00    2.00     0  cadvisor

02:43:53 PM   UID       PID    %usr %system  %guest    %CPU   CPU  Command
02:43:54 PM     0     32673    1.00    0.00    0.00    1.00     0  cadvisor
```

显示完整命令参数加-l：
```
[root@officeci001 ~]# pidstat -C cadvisor -l 1
Linux 3.10.0-957.21.3.el7.x86_64 (officeci001.pano.video) 	02/17/2020 	_x86_64_	(2 CPU)

02:44:38 PM   UID       PID    %usr %system  %guest    %CPU   CPU  Command
02:44:39 PM     0     32673    0.00    1.00    0.00    1.00     0  /usr/bin/cadvisor -logtostderr -port 18080

02:44:39 PM   UID       PID    %usr %system  %guest    %CPU   CPU  Command

02:44:40 PM   UID       PID    %usr %system  %guest    %CPU   CPU  Command
02:44:41 PM     0     32673    2.00    1.00    0.00    3.00     0  /usr/bin/cadvisor -logtostderr -port 18080
^C

Average:      UID       PID    %usr %system  %guest    %CPU   CPU  Command
Average:        0     32673    0.67    0.67    0.00    1.33     -  /usr/bin/cadvisor -logtostderr -port 18080
```

查看线程数据加-T ALL
```
[root@officeci001 ~]# pidstat -l -T ALL 1
Linux 3.10.0-957.21.3.el7.x86_64 (officeci001.pano.video) 	02/17/2020 	_x86_64_	(2 CPU)

02:45:59 PM   UID       PID    %usr %system  %guest    %CPU   CPU  Command
02:46:00 PM     0     32673    0.00    0.99    0.00    0.99     0  /usr/bin/cadvisor -logtostderr -port 18080

02:45:59 PM   UID       PID    usr-ms system-ms  guest-ms  Command
02:46:00 PM     0     32673         0        10         0  /usr/bin/cadvisor -logtostderr -port 18080

02:46:00 PM   UID       PID    %usr %system  %guest    %CPU   CPU  Command
02:46:01 PM     0     18358    0.00    1.00    0.00    1.00     1  pidstat -l -T ALL 1
02:46:01 PM     0     32673    1.00    0.00    0.00    1.00     0  /usr/bin/cadvisor -logtostderr -port 18080

```

显示上下文切换情况加-w：
```
[root@officeci001 ~]# pidstat -l -w 1
Linux 3.10.0-957.21.3.el7.x86_64 (officeci001.pano.video) 	02/17/2020 	_x86_64_	(2 CPU)

02:48:15 PM   UID       PID   cswch/s nvcswch/s  Command
02:48:16 PM     0         9     73.00      0.00  rcu_sched
02:48:16 PM     0        14      1.00      0.00  ksoftirqd/1
02:48:16 PM     0       352     20.00      0.00  xfsaild/nvme0n1
02:48:16 PM     0     10878      2.00      0.00  kworker/1:1
02:48:16 PM   999     12579     11.00      0.00  redis-server *:6379
02:48:16 PM  1008     17758      1.00      0.00  sshd: lipingmao@pts/2
02:48:16 PM     0     18465      4.00      0.00  kworker/0:0
02:48:16 PM     0     18567      1.00      1.00  pidstat -l -w 1
```

自愿上下文切换vs非自愿上下文切换：
自愿上下文切换  ：通常是等待IO（说明io多）
非自愿上下文切换：通常是时间片用完（说明cpu真忙）
```
              cswch/s
                     Total  number of voluntary context switches the task made per second.  A vol‐
                     untary context switch occurs  when  a  task  blocks  because  it  requires  a
                     resource that is unavailable.

              nvcswch/s
                     Total  number  of non voluntary context switches the task made per second.  A
                     involuntary context switch takes place when a task executes for the  duration
                     of its time slice and then is forced to relinquish the processor.
```

显示内存缺页情况可以加-r：
```
[root@officeci001 ~]# pidstat -r  1
Linux 3.10.0-957.21.3.el7.x86_64 (officeci001.pano.video) 	02/17/2020 	_x86_64_	(2 CPU)

02:53:39 PM   UID       PID  minflt/s  majflt/s     VSZ    RSS   %MEM  Command
02:53:40 PM     0     18958    132.00      0.00  108316   1064   0.03  pidstat

02:53:40 PM   UID       PID  minflt/s  majflt/s     VSZ    RSS   %MEM  Command
02:53:41 PM     0     18958    148.00      0.00  108316   1152   0.03  pidstat
02:53:41 PM     0     32217     16.00      0.00  109096   1352   0.04  containerd-shim
```

显示进程读写磁盘情况加-d：
```
[root@officeci001 ~]# pidstat -d 1
Linux 3.10.0-957.21.3.el7.x86_64 (officeci001.pano.video) 	02/17/2020 	_x86_64_	(2 CPU)

02:55:02 PM   UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s  Command
02:55:03 PM     0      6392      0.00     12.00      0.00  dockerd
02:55:03 PM     0      6576      0.00      8.00      0.00  java
02:55:03 PM     0     32656      0.00      4.00      4.00  containerd-shim

```

字段含义：
```
              kB_rd/s
                     Number of kilobytes the task has caused to be read from disk per second.

              kB_wr/s
                     Number of kilobytes the task has caused, or shall cause to be written to disk
                     per second.

              kB_ccwr/s
                     Number  of  kilobytes  whose  writing to disk has been cancelled by the task.
                     This may occur when the task truncates some dirty pagecache.  In  this  case,
                     some IO which another task has been accounted for will not be happening.
```


由此可见，pidstat可以获取进程级别CPU使用情况，上下文交换，disk读写大小，缺页情况等信息。分析进程级别资源非常有用。
