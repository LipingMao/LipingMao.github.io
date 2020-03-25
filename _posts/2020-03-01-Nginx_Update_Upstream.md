---
title: DevOps -- Nginx update upstream
---

> 不影响业务的升级nginx的Upstream几个办法。



**Option 1**: 使用API修改Upstream状态。（仅商业版支持）

**Option 2**: 使用healthcheck接口，控制upstream是否可用。



注意Option2中，如果upstream的健康检查没有通过，则会不让新的请求抓发，对于已经存在的请求，不会中断。



[1] https://www.nginx.com/blog/nginx-plus-backend-upgrades-individual-servers/
