---
title: DevOps -- Nginx 代理Gitlab SSH
---



## 问题描述

用户下载Gitlab代码时仅能通过HTTP访问，不能使用SSH下载代码。

## 问题原因

这是因为我们Gitlab由Nginx代理，HTTP被Gitlab代理，但是SSH没有被Nginx代理，直接访问ssh下载代码会访问到nginx所在的机器，而不是gitlab所在机器。

Nginx有tcp代理能力，但是Nginx所在的机器22端口已经被sshd占用，需要解决端口冲突。

## 解决方案

为了减少客户端配置，将nginx的ssh默认端口改为2222端口。Nginx代理gitlab的22端口。

ssh修改：

```
/etc/ssh/sshd_config
 
#Port 22
Port 2222
 
systemctl restart sshd
```

Nginx代理22端口配置如下：

```
stream {
upstream ssh{
        hash $remote_addr consistent;
        server  10.0.4.89:22 max_fails=3 fail_timeout=10s;
    }
 server{
        listen 22;
        proxy_connect_timeout 1h;
        proxy_timeout 1h;
        proxy_pass ssh;
    }
}
```

