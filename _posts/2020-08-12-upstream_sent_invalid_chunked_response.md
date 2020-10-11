---
title: DevOps  -- upstream sent invalid chunked response
---

Nginx 发现以下错误:
```
upstream sent invalid chunked response while reading upstream
```

最后发现是上游服务返回的response中带了chunked transfer encoding，这是HTTP1.1中才支持的feature，而nginx在proxy时默认情况下使用的是HTTP 1.0。因此需要在配置中显式指定HTTP 1.1:
```
proxy_http_version 1.1
```
