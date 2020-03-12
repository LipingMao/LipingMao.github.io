---
title: DevOps -- Cadvisor Memory Usage Metrics
---

> 使用Prometheus + cadvisor监控时，Prometheus里面的监控数据和用docker stats看到的数据有很大区别。最终发现在prometheus中收集的container_memory_usage_bytes并不是docker stats中的MEM USAGE。


以下是docker stats的输出：

```
[root@sh80dbstore002 ~]# docker stats
CONTAINER ID        NAME                                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PIDS
d69385e97f71        compassionate_hamilton              0.00%               852KiB / 1.795GiB     0.05%               0B / 0B             38.4MB / 0B         2
```

Prometheus目前收集的是以下数据：
```
container_memory_usage_bytes
container_memory_rss
container_memory_cache
```

目前，我们使用container_memory_usage_bytes用于表示容器内存使用情况，但是此处，并不是在docker stats中的MEM USAGE。container_memory_usage_bytes为container_memory_rss + container_memory_cache的值。而在docker stats中看到的是container_memory_rss的值。
