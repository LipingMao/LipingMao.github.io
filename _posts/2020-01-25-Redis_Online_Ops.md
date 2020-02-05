---
title: DevOps -- Redis in production
---

Redis 线上环境应该禁止使用以下命令：
```
KEYS *
FLUSHALL
FLUSHDB
```

Refer: https://blog.csdn.net/bntX2jSQfEHy7/article/details/84207884
