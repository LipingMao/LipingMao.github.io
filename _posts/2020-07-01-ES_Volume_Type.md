---
title: DevOps -- ES on AWS
---

### 问题：

ES在AWS的data盘使用st1之后，iowait变高。

### 原因：

st1 和 sc1 卷的性能模型针对顺序 I/O 进行了优化，支持高吞吐量工作负载，对具有混合 IOPS 和吞吐量的工作负载提供可接受的性能，不建议使用具有小型随机 I/O 的工作负载。
例如，1 MiB 或更小的 I/O 请求计为 1 MiB I/O 积分。但是，如果是顺序 I/O，则会合并为 1 MiB I/O 数据块，并且只计为 1 MiB I/O 积分。
我们的ES的data盘，在0.5T的时候，有的积分是20每秒。而ES节点的随机读写在100-150之间，因此在耗费完积分之后，ES的磁盘会被限制。
因此在目前容量下，可以使用gp2类型的磁盘，等ES的data盘比较大时，可以评估是否可以用st1类型的磁盘。



https://docs.amazonaws.cn/AWSEC2/latest/UserGuide/ebs-volume-types.html#inefficiency
