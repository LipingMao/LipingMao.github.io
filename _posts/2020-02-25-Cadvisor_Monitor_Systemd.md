---
title: DevOps -- Cadvisor Monitor Systemd
---



> 我们大多数的应用是容器化的，但是也有一些应用没有容器化，这些应用使用了Systemd管理。之前一直以外Cadvisor只能监控容器的资源使用情况，实际上Cadvisor也可以监控Systemd管理的非容器的资源使用状况。



 可以看到每个id为system.slice/XXX.service可以获取到CPU/Memory等信息：

![image-20200313092713581](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture//image-20200313092713581.png)



可以通过systemd-cgtop查看systemd相关的资源使用情况：

```
Path                                                          Tasks   %CPU   Memory  Input/s Output/s

/                                                                47    4.1     1.4G        -        -
/system.slice                                                     -    1.5     1.2G        -        -
/docker                                                           -    1.3   172.0M        -        -
/system.slice/cloud-final.service                                10    1.2   655.0M        -        -
/user.slice                                                       6    1.2    55.1M        -        -
/docker/c434e38970a...e8f6726c8e7810e89304a1f36be2945ead683       1    0.7    10.5M        -        -
/docker/7e8bac7681c...27d4f4a43ec01453d106908ff3fdc00126a1a       1    0.6    74.7M        -        -
/system.slice/media_server.service                                1    0.2    14.4M        -        -
/system.slice/containerd.service                                  8    0.0    84.2M        -        -
/system.slice/docker.service                                      1    0.0    56.8M        -        -
/docker/88002656c80...54b7bfb58442c1db7fd7123f15bd77b86b3f0       1    0.0    12.7M        -        -
/system.slice/tuned.service                                       1    0.0    11.2M        -        -
/system.slice/ntpd.service                                        1    0.0   900.0K        -        -
/system.slice/rsyslog.service                                     1    0.0   353.0M        -        -
/docker/2d5ab31f687...0e9b1245d3da3fff33efd488392415312f99c       1      -    21.0M        -        -
/docker/556f7fdc5b3...4389e3edbc71f7ac1a9a5af1df0b6fd106362       1      -    11.1M        -        -
/docker/9c50b4bf6de...73c87167198641e8d91128b71f70d568e18c0       4      -    19.5M        -        -
/docker/e7cbb0d3346...b46ca5a6a7ef85dd2cca59cfbed1b12a27010       1      -     1.4M        -        -
/system.slice/acpid.service                                       1      -   140.0K        -        -
/system.slice/atd.service                                         1      -   240.0K        -        -
/system.slice/auditd.service                                      1      -   560.0K        -        -
/system.slice/crond.service                                       1      -     1.1M        -        -
/system.slice/dbus.service                                        2      -     3.3M        -        -
/system.slice/libstoragemgmt.service                              1      -   172.0K        -        -
/system.slice/lvm2-lvmetad.service                                1      -   344.0K        -        -
/system.slice/network.service                                     1      -     1.9M        -        -
/system.slice/polkit.service                                      1      -     7.2M        -        -
/system.slice/sshd.service                                        1      -     5.8M        -        -
/system.slice/system-getty.slice                                  1      -   128.0K        -        -
/system.slice/system-getty.slice/getty@tty1.service               1      -        -        -        -
/system.slice/system-serial\x2dgetty.slice                        1      -   128.0K        -        -
/system.slice/syste...etty.slice/serial-getty@ttyS0.service       1      -        -        -        -
/system.slice/systemd-journald.service                            1      -    42.8M        -        -
/system.slice/systemd-logind.service                              1      -   976.0K        -        -
/system.slice/systemd-udevd.service                               1      -   800.0K        -        -
/user.slice/user-0.slice/session-257039.scope                     6      -        -        -        -
```



这样cadvisor就可以统一监控容器和systemd相关的服务。

