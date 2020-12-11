---
title: DevOps -- filebeat delay
---

我们使用filebeat收集log，metrics，billing，event。在一些场景下，我们对收集的数据时间要求较高。而通过日志方式收集数据，再发送到中心统一处理的pipeline较长。
整个链路类似：
```
Logs -> filebeat -> Media Kafka -> Kafka Mirror Maker -> Central Kafka -> Kafka Stream -> App
```

我们发现数据有时会随机性延迟，debug定位发现，主要延迟发生在filebeat获取到日志的这个阶段。
有以下参数影响收集时间：
```
backoff
The backoff options specify how aggressively Filebeat crawls open files for updates. You can use the default values in most cases.

The backoff option defines how long Filebeat waits before checking a file again after EOF is reached. The default is 1s, which means the file is checked every second if new lines were added. This enables near real-time crawling. Every time a new line appears in the file, the backoff value is reset to the initial value. The default is 1s.


max_backoff
The maximum time for Filebeat to wait before checking a file again after EOF is reached. After having backed off multiple times from checking the file, the wait time will never exceed max_backoff regardless of what is specified for backoff_factor. Because it takes a maximum of 10s to read a new line, specifying 10s for max_backoff means that, at the worst, a new line could be added to the log file if Filebeat has backed off multiple times. The default is 10s.

Requirement: Set max_backoff to be greater than or equal to backoff and less than or equal to scan_frequency (backoff <= max_backoff <= scan_frequency). If max_backoff needs to be higher, it is recommended to close the file handler instead and let Filebeat pick up the file again.


backoff_factor
This option specifies how fast the waiting time is increased. The bigger the backoff factor, the faster the max_backoff value is reached. The backoff factor increments exponentially. The minimum value allowed is 1. If this value is set to 1, the backoff algorithm is disabled, and the backoff value is used for waiting for new lines. The backoff value will be multiplied each time with the backoff_factor until max_backoff is reached. The default is 2.
```

如果event数据，没有经常产生，读取时就会产生EOF，而filebeat在读取到EOF时，会backoff一段时间，这个延迟可能会是1-10s之间。
因此filebeat产生了延迟。
