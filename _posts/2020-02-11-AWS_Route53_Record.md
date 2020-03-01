---
title: DevOps -- Resolve route53 record in China
---

在国内有时会出现解析AWS Route53 DNS记录失败的情况。
我们将ttl从60s改为300s后，出现几率大大降低。但目前没有找到Root Cause。推测提高ttl减少了运营商中间节点对dns请求，减少了失败率。
