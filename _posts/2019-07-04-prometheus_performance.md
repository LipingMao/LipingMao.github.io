---
title: Prometheus Performance
---

## Background

> One natual question start to use prometheus is how the performance it looks like. I searched some doc and tried in our environment to a little bit understand how it looks like.

## Performance

This [blog](https://www.percona.com/blog/2018/09/20/prometheus-2-times-series-storage-performance-analyses/), describe how the performance looks like. 
In short: 

> 200k sample/sec need 1 cpu core and 16G ram

I tested in our environment, here is the performance in our environment:

> 150k sample/sec used about 1.8 cpu and 14G ram

So , let's say in our real usage environment, it is about :

> 100k sample/sec  need 1 cpu , 8G ram

## Do not over use label

The [best practices](https://prometheus.io/docs/practices/instrumentation/#do-not-overuse-labels) is not over use label. Do not use unscope resource in label, each k-v will be one time series db. We should avoid this situation.
