---
title: DevOps -- NTPD not started
---

### 问题内容

在AWS环境的CentOS7安装了NTPD并且systemctl enable ntpd之后，在系统重启后ntpd依然不能启动。


### 问题原因

由于系统启动了chronyd, 会导致ntpd无法启动，可以禁用chronyd。

```
systemctl stop chronyd; systemctl disable chronyd; systemctl enable ntpd;systemctl restart ntpd

```
