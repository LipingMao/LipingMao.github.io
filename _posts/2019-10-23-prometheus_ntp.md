---
title: 使用Prometheus监控时间指标
---


> 我们使用Prometheus作为监控系统，我们希望当系统发生时间漂移时可以收到报警，当系统发现和上游NTP服务器之间抖动较大时，可以及时收到通知。



# Prometheus 原生监控NTP

Node_exporter支持对于ntp的监控，代码实现非常简单[1]，代码模拟了一个SNTP客户端，对配置的ntp server发送一个请求包进行请求。在文档[2]中比较详细的记录了相关metrics。但是测试我们发现几个指标并不符合我们目前需求，比如: `node_ntp_offset`, 这个指标仅能反应SNTP与本地机器之间的offset，并非Server与NTP服务器之间的offset，看起来并没有太大作用。node_exporter默认监控的是127.0.0.1 local的ntp服务，如果将其改为Server使用的NTP服务器是否就能解决问题呢？ 这样做并不是特别好，原因如下：

1) 并非所有NTP Server都能正确对SNTP反应，比如腾讯云的内网NTP就不能响应SNTP。

2) 对NTP Server大大增加了流量。 NTP稳定后，一般请求周期是1024s，如果使用这种方式，则会每个prometheus server拉取数据时，都会有NTP请求，比原来NTP请求多很多。

基于上述原因，[1]并不能满足要求。



# Prometheus text collector

Prometheus可以支持text collector，顾名思义，可以将metrics信息放到一个约定好的目录，prometheus text collector就可以把metrics暴露出来。至于如何收集这些metrics，可以使用外部脚本去做。社区有一些现成的脚本[3]，其中ntpd_metrics.py[4]可以满足我们的需求。仅需要周期性运行此脚本更新到一个事先约定好的目录即可。

使用text collector有一些Pros and Cons。

**Pros：** 

对node_exporter源码没有侵入性，后期单独升级node_exporter对此部分逻辑没有影响。

**Cons：**

text collector脚本的维护在宿主机上并不优雅，可以考虑将所有text collector的脚本放在一个容器里面，将结果交给Node Exporter容器（如果node exporter是使用容器化的话）



# 关键指标和告警

```
# HELP ntpd_peer_status NTPd metric for peer_status
# TYPE ntpd_peer_status gauge
ntpd_peer_status{remote="169.254.0.2",reference="100.122.36.196",stratum="2",type="unicast"} 6.000000
# HELP ntpd_delay_milliseconds NTPd metric for delay_milliseconds
# TYPE ntpd_delay_milliseconds gauge
ntpd_delay_milliseconds{remote="169.254.0.2",reference="100.122.36.196"} 0.690000
# HELP ntpd_offset_milliseconds NTPd metric for offset_milliseconds
# TYPE ntpd_offset_milliseconds gauge
ntpd_offset_milliseconds{remote="169.254.0.2",reference="100.122.36.196"} -1.580000
# HELP ntpd_jitter_milliseconds NTPd metric for jitter_milliseconds
# TYPE ntpd_jitter_milliseconds gauge
ntpd_jitter_milliseconds{remote="169.254.0.2",reference="100.122.36.196"} 0.789000
# HELP ntpd_rootdelay NTPd metric for rootdelay
# TYPE ntpd_rootdelay gauge
ntpd_rootdelay 29.972000
# HELP ntpd_rootdisp NTPd metric for rootdisp
# TYPE ntpd_rootdisp gauge
ntpd_rootdisp 40.797000
# HELP ntpd_offset NTPd metric for offset
# TYPE ntpd_offset gauge
ntpd_offset -1.580000
# HELP ntpd_sys_jitter NTPd metric for sys_jitter
# TYPE ntpd_sys_jitter gauge
ntpd_sys_jitter 0.000000
```



ntpd_offset_milliseconds： 代表了local与时间服务器之间的offset，当大于128ms时，考虑进行报警。NTP一般能保证差值不大于128ms。

ntpd_jitter_milliseconds：代表了local到ntpd server的抖动，大于20ms可进行报警，如果内网NTP的话，一般此值为1左右。

从这些参数，我们可以知道服务器local发生时间跳变的情况，ntp Server稳定性情况等等信息。





[1] https://github.com/prometheus/node_exporter/blob/master/collector/ntp.go

[2] https://github.com/prometheus/node_exporter/blob/master/docs/TIME.md

[3] https://github.com/prometheus-community/node-exporter-textfile-collector-scripts

[4] https://github.com/prometheus-community/node-exporter-textfile-collector-scripts/blob/master/ntpd_metrics.py





