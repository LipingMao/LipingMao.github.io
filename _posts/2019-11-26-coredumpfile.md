---
title: DevOps -- Coredump file
---

> 应用调试发生异常退出时，发现腾讯云和AWS上面的主机没有输出coredump文件。

Linux默认的ulimit是不会生成coredump文件的：
```
# ulimit -a
core file size          (blocks, -c) 0                 <--- Core dump 默认为0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 7269
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 100001
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 7269
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

因此可以通过ulimit -c unlimit将core file size设置为不限制，则可以输出到根目录下。
```
# ulimit -c unlimited
# ulimit -c
unlimited
```



输出的目录是由以下core_pattern配置控制，默认情况下生成到根目录：
```
# cat /proc/sys/kernel/core_pattern
core
```

在容器中已经默认打开core file，这是因为docker启动时利用systemd配置了limit，容器会继承docker的配置：
```
# cat /usr/lib/systemd/system/docker.service
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
BindsTo=containerd.service
After=network-online.target firewalld.service containerd.service
Wants=network-online.target
Requires=docker.socket
 
[Service]
...
 
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity               <--- unlimited
 
# Comment TasksMax if your systemd version does not support it.
# Only systemd 226 and above support this option.
TasksMax=infinity
 
...
 
[Install]
WantedBy=multi-user.target
```

以GSLB的容器为例，可以看到core file是unlimited：
```
[root@sh01gslb001 ~]# docker exec -it 43 sh
sh-4.2# ulimit -a
core file size          (blocks, -c) unlimited
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 7269
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1048576
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) unlimited
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```


