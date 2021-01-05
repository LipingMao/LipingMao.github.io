---
title: DevOps -- filebeat delay
---



> filebeat是一个Elastic开源的轻量级收集日志的agent，在收集日志过程中会有一些延迟的场景。本文分析了一些延迟原因。



一般日志收集时，使用一个agent，可以是filebeat/logstash agent采集日志，然后发送到Kafka中，之后Logstash从kafka中读出数据，写入ElasticSearch集群。数据流如下：

```
filebeat(logstash) --> Kafka --> Logstash --> ElasticSearch
```



作为日志收集的agent，filebeat和logstash agent各有好坏。一般来说filebeat对比logstash更加轻量，占用资源更少，而logstash拥有更加丰富的filter功能。在不需要很多强大filter功能的情况下，使用filebeat是一个不错的选择。



然而，在使用filebeat过程中，发现日志延迟有不稳定的情况出现。有时日志延迟1s，而有时会延迟超过10s才从filebeat发送到kafka集群。这是什么原因呢？



要找到问题，需要先理解一下filebeat是如何工作的。

1)  filebeat会定周期扫描文件路径，发现文件的变化。

2）为一个文件创建一个harvester，读取文件内容，将内容存放到队列中。由output从队列中取出，发往后续处理。（后续可以是Kafka， es，logstash，redis等不同的output）



因此可能影响发送速度的有：

1） 发现文件变化周期的配置选项。

2） harvester读取文件相关配置。例如：读取到EOF时的backoff和max_backoff，当文件没有更新时的inactive timeout等。