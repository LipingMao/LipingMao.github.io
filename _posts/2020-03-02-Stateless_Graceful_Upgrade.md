---
title: DevOps -- Stateless Application Gracefully upgrade
---



> 无状态应用如果升级时直接重启，也会导致正在处理的请求出现问题，如果用户的请求量不小，最好可以避免这类问题。



我们使用nginx upstream check module[1]主动探测。当检测到health接口时返回200时，认为是OK的：

```
        upstream cluster {

            # simple round-robin
            server 192.168.0.1:80;
            server 192.168.0.2:80;

            check interval=3000 rise=2 fall=5 timeout=1000 type=http;
            check_http_send "HEAD / HTTP/1.0\r\n\r\n";
            check_http_expect_alive http_2xx http_3xx;
        }
```



每个服务提供health和管理health两个接口，例如：

```
/health   200 OK / 500 Failed
/mgmt     Admin up / Admin down 控制health接口
```



Ansible的升级主要逻辑如下：

```
对于每一个node：
    1. 调用管理接口admin down服务。
    2. 等待一段时间，直到Nginx认为服务是down的，此时Nginx并不会关闭之前连接，仅仅会不新调度连接。
    3. 等待timeout时间，业务处理完当前请求。
    4. 升级node，此时没有连接，不会影响。
    5. 重启服务后，等待nginx upstream 状态恢复。
```





[1] https://github.com/yaoweibin/nginx_upstream_check_module






