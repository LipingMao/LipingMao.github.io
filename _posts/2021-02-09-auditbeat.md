---
title: DevOps -- auditbeat
---

> auditbeat是轻量级审计agent


目前audit beat可以支持以下三个module:
```
Auditd
File Integrity
System
```

* Auditd可以接收Linux Audit Framework的events。
* file_integrity可以检测文件是否发生变化
* System可以检测一些系统信息,具体如下: 

```
    - host    # General host information, e.g. uptime, IPs
    - login   # User logins, logouts, and system boots.
    - process # Started and stopped processes
    - socket  # Opened and closed sockets
    - user    # User information
```


