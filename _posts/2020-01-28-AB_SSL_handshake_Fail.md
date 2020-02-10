---
title: DevOps -- ab ssl handshake fail
---


ab测试时有很多以下错误：
```
SSL handshake failed (5).
SSL handshake failed (5).
SSL handshake failed (5).
SSL handshake failed (5).
SSL handshake failed (5).
SSL handshake failed (5).
SSL handshake failed (5).
SSL handshake failed (5).
```

检查被测试对象的证书没有问题，最后发现是测试的并发过高。测试对象产生了IO相关的错误（返回值5）。降低测试并发就没问题。

https://serverfault.com/questions/563412/apache-bench-ssl-handshake-failing-directly-related-to-concurrency-level
