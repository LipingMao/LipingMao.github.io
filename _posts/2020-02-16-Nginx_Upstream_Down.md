---
title: DevOps -- Nginx Upstream Down
---

> Nginx在处理上游服务器down的时候的动作是和Haproxy不太一样的，Nginx默认使用“被动”检测的方式。本文记录了一些需要注意的地方。

假设Upstream有VM1:Port1， VM2:Port2, 在Failover时，有两种情况：
```
1. 服务Down
2. Server Down
```

Nginx failover的机制是，你可以配置类似以下两个选项：
```
max_fails       ：最多多少次失败，默认是1
fail_timeout    ：失败后多久再重试，默认是10s
```

大概的机制是：
转发到Upstream，如果发现服务是Down的，操作系统会不存在这个端口，很快给Nginx相应，Nginx会知道这台机器不能相应，然后转发到其他服务器。并且在10s内不再转发到这台服务器。
从这个机制，可以看到Nignx使用的是Lazy的方式，Nignx没有主动探测服务器是否可用，而是利用转发请求，如果有错误则静默一段时间。

这个机制在服务Down的时候没有问题，但是到Server Down的时候就有问题了。 如果服务器VM1 down了，就有问题了，因为Nginx并不知道是因为Server Down 还是 因为服务器响应慢。
因此当Server Down时，Nginx就有几率转发到一台Down的VM，并且这个API响应延迟非常久。
通过调整一些参数。可以减少影响，例如降低响应超时时间，增加静默时间等，但是这些方式并没有特别好的解决这个问题。因为都会产生一定周期内一两次请求延迟变的很大。

Nginx的商业版中支持主动探测的功能，在开源版中nginx_upstream_check_module模块可以实现类似效果。
参考[1][2]可以编译这个module并且配置，此处不在赘述。


Refer:
[1] https://www.cnblogs.com/handsomeye/p/9604785.html
[2] https://github.com/yaoweibin/nginx_upstream_check_module
