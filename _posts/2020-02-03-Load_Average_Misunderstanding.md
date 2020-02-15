---
title: DevOps -- Load Average Misunderstanding
---



> 人们总是对常见的事物很熟悉，但是很少能明确说出它到底指的是什么？比如Load Average。本文从几个问题出发，聊一聊Load Average容易被误解的地方。



### Q1. 什么是Load Average？

当有性能问题，习惯性的打开top，你会在右上角有三个数字，如下所示：

![image-20200215154243131](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/image-20200215154243131.png)

图中 load average： 0.08  0.07  0.06分别表示1分钟，5分钟，15分钟的Load Average。



这个数字代表什么意思呢？ Load Average表示可运行状态和不可中断状态的task的“平均”。

通俗点讲这个值体现的是一个队列长度，这个队列有可以被调度的Tasks，以及不可中断状态（如等待IO，lock等）的Task的“平均”队列长度。 



举个例子来理解（例子引用自[2]）：

如果把Tasks看成是汽车，一个cpu core看作是一座桥，不同的load average就类似以下情况

![image-20200215183048626](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/image-20200215183048626.png)

图中：

1.00 就相当于汽车正好达到桥的容量。

0.50相当于这座桥还能有一半的容量。

1.70相当于已经过载，需要排队。



同样的，在多核的情况下，就可以想象成多个车道。在比较不同CPU核数的Load Average时，通常可以归一化，即Load Average / CPU Num



### Q2. 为什么要有1分钟，5分钟，15分钟三个值呢？



这是因为这三个值分别体现了不同时间段内的负载情况，可以快速判断出系统近期的负载变化情况：

如果三个值接近，则说明近期系统load平稳。

如果1分钟>5分钟>15分钟，则说明系统load在增加，反之亦然。





### Q2. 我们常说的CPU Load Average，这个叫法准确么？



从定义看出：load是可运行状态，不可中断状态的task的总和。

可运行状态比较好理解，那什么是不可中断（Uninterruptible）状态呢？

出于不可中断状态的task通常需要避免中断，这样的task常见的场景是出于Block IO或者lock中，如果你用Top或者PS可以看到这些程序是在“D”状态。



在我们理解了Load Average是这么定义之后，理解一下为什么Linux要这么定义呢？

如果Load Average单纯是表示CPU的Load情况，应该只使用可运行状态来体现CPU的繁忙程度。但是Linux中还计算了不可中断状态（wait IO，lock等）的task，这些task并没有体现CPU是否繁忙，而是体现了其他资源的繁忙程度。



Netflix的性能优化大师 [Brendan Gregg](http://www.brendangregg.com/blog/index.html) 在[1]中为我们找到了答案。

最初Linux仅仅计算可运行状态的task，1993年以下同学修改了这个逻辑，从注释中可以看出，他希望Load Average可以综合的体现出系统的繁忙程度，而不是单单CPU是否繁忙，因此在load average中加入了Uninterruptible状态的计算：

 ```
From: Matthias Urlichs <urlichs@smurf.sub.org>
Subject: Load average broken ?
Date: Fri, 29 Oct 1993 11:37:23 +0200


The kernel only counts "runnable" processes when computing the load average.
I don't like that; the problem is that processes which are swapping or
waiting on "fast", i.e. noninterruptible, I/O, also consume resources.

It seems somewhat nonintuitive that the load average goes down when you
replace your fast swap disk with a slow swap disk...

Anyway, the following patch seems to make the load average much more
consistent WRT the subjective speed of the system. And, most important, the
load is still zero when nobody is doing anything. ;-)

--- kernel/sched.c.orig Fri Oct 29 10:31:11 1993
+++ kernel/sched.c  Fri Oct 29 10:32:51 1993
@@ -414,7 +414,9 @@
    unsigned long nr = 0;

    for(p = &LAST_TASK; p > &FIRST_TASK; --p)
-       if (*p && (*p)->state == TASK_RUNNING)
+       if (*p && ((*p)->state == TASK_RUNNING) ||
+                  (*p)->state == TASK_UNINTERRUPTIBLE) ||
+                  (*p)->state == TASK_SWAPPING))
            nr += FIXED_1;
    return nr;
 }
--
Matthias Urlichs        \ XLink-POP N|rnberg   | EMail: urlichs@smurf.sub.org
Schleiermacherstra_e 12  \  Unix+Linux+Mac     | Phone: ...please use email.
90491 N|rnberg (Germany)  \   Consulting+Networking+Programming+etc'ing      42
 ```





因此在Linux中，“CPU Load Average”是不准确的，应该叫System Load Average，因为它较为综合的体现了系统task的繁忙程度，而不只是单纯的CPU负载。



### Q3. Load Average = 1是指“1个task 使用1 个CPU 100%”么？



理解了Load Average的定义后，那Linux是如何计算出过去1分钟/5分钟/15分钟的值的“平均”值呢？

Linux使用了阻尼算法平滑计算过去一段时间的Load Average。



简单看一下计算的相关代码（参考[3]中具体讨论了计算算法）：

```c
https://github.com/torvalds/linux/blob/master/include/linux/sched/loadavg.h

#define LOAD_FREQ	(5*HZ+1)	/* 5 sec intervals */
#define EXP_1		1884		/* 1/exp(5sec/1min) as fixed-point */
#define EXP_5		2014		/* 1/exp(5sec/5min) */
#define EXP_15		2037		/* 1/exp(5sec/15min) */
```

可以看到计算Load Average是以5秒为间隔进行计算的，这里计算1分钟/5分钟/15分钟有三个Magic Number EXP_N，从注释中可以看到是为了加速1/exp(5sec/N min)的计算。

然后再计算当前Load时，使用EXP_N对当前值进行平滑修正：

```c
https://github.com/torvalds/linux/blob/master/kernel/sched/loadavg.c

void calc_global_load(unsigned long ticks)
{
...

	avenrun[0] = calc_load(avenrun[0], EXP_1, active);
	avenrun[1] = calc_load(avenrun[1], EXP_5, active);
	avenrun[2] = calc_load(avenrun[2], EXP_15, active);

...
}
```

 

正因为这种阻尼计算的方式，如果有1个task使用了一个100%使用了一个CPU，会对load average产生类似以下曲线（图片出自[1]）:

![image-20200215192323649](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/image-20200215192323649.png)

从图中可以看到，如果运行了60s的一个task，会让1 min Load Average达到0.62左右，当运行了大概300s后，1min Load Average会达到1。同理让5 min / 15 min Load Average达到1需要更长的时间。



### Q4. Load Average高就只是CPU使用率高么？还有哪些常见原因？



说了这么多，常见哪些原因会导致Load Average高呢？

在理解了Load Average的定义后，我们发现这个问题就等价于：哪些原因会导致处于 “可运行状态”的task 和 “不可中断状态”的task增高。你会发现不论是CPU / 内存 / 磁盘 / 网络使用率高，都有可能造成Load Average上升。

下面是几种常见的情况：

* 存在CPU密集型应用。
* 有过多的可运行状态的Task造成相互抢占，造成上下文切换高。
* 内存频繁访问。
* 内存不足，SWAP变高。
* Disk IO高，程序处于等待IO状态。
* 网络流量大，内核软中断（si）高。

每种情况需要使用不同的性能检测工具来定位真正的问题点。对于这些情况的具体分析，本文不再展开。

但是，下次当Load Average再升高时， 除了使用top盯着看以外，思路上可以看看，是处于哪种的情况 ;-)



### 后记

1. 一个常见的概念往往因为太常见，而忽略它本身的含义。 

2. Brendan Gregg对问题深究的精神令人钦佩，为了得到答案，查找到一个27年前Linux的修改记录，要知道当时Linux的代码还不在github管理，修改记录是从2008年的http://oldlinux.org/Linux.old/mail-archive/中找到的。



### Refer

[1] http://www.brendangregg.com/blog/2017-08-08/linux-load-averages.html

[2] https://scoutapm.com/blog/understanding-load-averages

[3] https://www.helpsystems.com/resources/guides/unix-load-average-part-1-how-it-works



