---
title: DevOps -- cAdvisor key metrics
---

> cAdvisor几个有用的metrics。


> container_cpu_cfs_throttled_seconds_total

如果限制了容器CPU使用，这个metrics反映了cpu被限制的情况。

> container_processes

容器内进程/线程的数量。 

> container_cpu_usage_seconds_total

容器内CPU使用情况。

> container_fs_reads_bytes_total / container_fs_reads_total / container_fs_writes_bytes_total / container_fs_writes_total

容器文件系统读/写的次数/bytes。

> container_memory_rss

容器rss使用情况。
