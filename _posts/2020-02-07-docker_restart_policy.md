---
title: DevOps -- Docker Restart Policy 
---

容器有以下重启策略：
```
no，默认策略，在容器退出时不重启容器
on-failure，在容器非正常退出时（退出状态非0），才会重启容器
on-failure:3，在容器非正常退出时重启容器，最多重启3次
always，在容器退出时总是重启容器
unless-stopped，在容器退出时总是重启容器，但是不考虑在Docker守护进程启动时就已经停止了的容器
```

systemd也有类似的策略：
```
Restart=always: 只要不是通过systemctl stop来停止服务，任何情况下都必须要重启服务，默认值为no
RestartSec=5: 重启间隔，比如某次异常后，等待5(s)再进行启动，默认值0.1(s)
StartLimitInterval: 无限次重启，默认是10秒内如果重启超过5次则不再重启，设置为0表示不限次数重启
```
