---
title: DevOps -- beats
---

Beats 是用go写的轻量级的用于Collect, Parse, and Ship的组件。
elastic做了很好的抽象，大部分通用逻辑都在libbeats中。

常见以下beats(除了这些还有社区beats)
```
Auditbeat Reference [7.10] — other versions
Filebeat Reference [7.10] — other versions
Functionbeat Reference [7.10] — other versions
Heartbeat Reference [7.10] — other versions
Journalbeat Reference [7.10] — other versions
Metricbeat Reference [7.10] — other versions
Packetbeat Reference [7.10] — other versions
Winlogbeat Reference [7.10] — other versions
```

[1] https://www.elastic.co/guide/en/beats/libbeat/current/beats-reference.html
