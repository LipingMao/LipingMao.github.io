---
title: DevOps -- HTTP Security Header
---

> HTTP有一些安全头部，本文简单列举了一些。

### Strict-Transport-Security

这个HTTP头表明网站仅支持HTTPS。一般网站如果使用https时，会对http发出301/302重定向，有了HSTS后，浏览器会自动强制https，少了一次call。例子如下：

> strict-transport-security: max-age=16070400; includeSubDomains

includeSubDomains表示同样作用于子域名。浏览器会自动重定向http到https。


### X-Frame-Options

> x-frame-options: SAMEORIGIN

DENY：不允许被任何页面嵌入
SAMEORIGIN：不允许被本域以外的页面嵌入
ALLOW-FROM uri：不允许被指定的域名以外的页面嵌入


### X-XSS-Protection

0：禁用XSS保护；
1：启用XSS保护；
1; mode=block：启用XSS保护，并在检查到XSS攻击时，停止渲染页面

X-Content-Type-Options


### X-Content-Type-Options

> X-Content-Type-Options: nosniff

有的资源没有定义Content-Type时，浏览器会猜测资源类型，这个例子是禁用浏览器的类型猜测行为。


### 目前配置

目前我们启用了以下安全头部：
```
x-content-type-options: nosniff
x-frame-options: SAMEORIGIN
x-xss-protection: 1; mode=block
```
