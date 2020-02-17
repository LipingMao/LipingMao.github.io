---
title: DevOps -- vmstat
---

> vmstat 可以帮助快速定位系统各种资源的关键参数。本文具体看以下一些参数：


以下为示例输出：
```
root@ip-172-31-0-243:~# vmstat 1
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 0  0      0 153060 128764 482468    0    0     8    22    1    2  0  0 99  0  0
 0  0      0 153060 128764 482468    0    0     0     0   51  228  0  0 100  0  0
 0  0      0 153060 128764 482468    0    0     0     0   44  206  0  0 100  0  0
 0  0      0 153060 128764 482468    0    0     0     0   40  206  0  0 100  0  0
 0  0      0 153060 128764 482468    0    0     0     0   44  206  0  0 100  0  0
 0  0      0 153060 128764 482468    0    0     0     0   42  208  0  0 100  0  0
 0  0      0 153060 128764 482468    0    0     0     0   40  208  0  0 100  0  0
```

其中：
vmstat 1 表示间隔1秒钟输出一行，全部输出的第一行为开机以来的历史数据，之后的是执行命令后间隔时间内的数据。



具体字段含义如下：

```
Procs
       r: The number of runnable processes (running or waiting for run time).
       b: The number of processes in uninterruptible sleep.

   Memory
       swpd: the amount of virtual memory used.
       free: the amount of idle memory.
       buff: the amount of memory used as buffers.
       cache: the amount of memory used as cache.
       inact: the amount of inactive memory.  (-a option)
       active: the amount of active memory.  (-a option)

   Swap
       si: Amount of memory swapped in from disk (/s).
       so: Amount of memory swapped to disk (/s).

   IO
       bi: Blocks received from a block device (blocks/s).
       bo: Blocks sent to a block device (blocks/s).

   System
       in: The number of interrupts per second, including the clock.
       cs: The number of context switches per second.
       
   CPU
       These are percentages of total CPU time.
       us: Time spent running non-kernel code.  (user time, including nice time)
       sy: Time spent running kernel code.  (system time)
       id: Time spent idle.  Prior to Linux 2.5.41, this includes IO-wait time.
       wa: Time spent waiting for IO.  Prior to Linux 2.5.41, included in idle.
       st: Time stolen from a virtual machine.  Prior to Linux 2.6.11, unknown.
```



当Procs中 

r比较大时，可以考虑是Ready的task比较多，造成可运行状态的task竞争。

b表示不可中断状态的task多，需要进一步看是不是IO多，还是lock比较多。



如果出现Swap，则考虑内存是否不足。



System：

in高，则说明中断过多，需要进一步查看什么中断导致过多。

cs高，则是上下文切换多，看看是那个进程/线程主要发生的切换。



CPU：

us高，是用户态程序消耗的多，可能优化程序算法。

sy高，是内核态消耗的多。

wa高，重点关注Disk IO相关内容。

st高，可能是虚拟化场景底层overcommit多。                  