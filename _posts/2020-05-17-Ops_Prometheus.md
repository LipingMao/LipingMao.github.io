---
title: DevOps -- Prometheus Staleness 
---


在prometheus federation后，对于stale的time series，有5分钟左右的延迟。
这是Prometheus staleness机制：https://prometheus.io/docs/prometheus/latest/querying/basics/#staleness
而默认的配置中query.lookback-delta为5分钟。
