---
title: DevOps -- AWS EC2 Instance Resize volume disk online
---

## 问题描述：

我们的业务支持系统部署在AWS EC2机器上，最初成本考虑使用了30G Disk的磁盘，随着Gitlab/Confluence等系统的使用，磁盘使用率已经超过了75%，有扩展磁盘的需求。

```
[root@ip-10-0-4-89 data]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/nvme0n1p1   30G   24G  7.0G  77% /
devtmpfs        7.7G     0  7.7G   0% /dev
tmpfs           7.7G   16K  7.7G   1% /dev/shm
tmpfs           7.7G  786M  6.9G  11% /run
tmpfs           7.7G     0  7.7G   0% /sys/fs/cgroup
tmpfs           1.6G     0  1.6G   0% /run/user/1000
overlay          30G   24G  7.0G  77% /var/lib/docker/overlay2/d8046f52b187b6c7474213deb8f0bd14b956814d69b1d33d552aecd2f19cabdc/merged
shm              64M     0   64M   0% /var/lib/docker/containers/937a86784983ab3cafc9bffe1e51d81e459bf4a0f9648f2e4d0a7a2053ef366f/shm
```





## 如何解决：

获取VM对应的EBS ID：

![image-20200205213013826](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/image-20200205213013826.png)



修改EBS卷的大小到需要的Size（⚠️ EBS仅支持扩大，注意备份）

![image-20200205213033114](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/image-20200205213033114.png)

等待EBS卷完成更新：

![image-20200205213052283](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/image-20200205213052283.png)

查看文件大小，通过growpart /dev/nvme0n1 1 扩展磁盘:

![image-20200205213108886](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/image-20200205213108886.png)

使用xfs_growfs -d / 扩展文件系统：

![image-20200205213127389](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/image-20200205213127389.png)



[1] https://docs.amazonaws.cn/AWSEC2/latest/UserGuide/recognize-expanded-volume-linux.html