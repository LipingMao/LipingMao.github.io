---
title: DevOps -- Interface Name
---

### 问题：

默认情况下，AWS build的VM主机的interface接口为ens5。 我们希望固定成eth0.





### 解决办法：

在Packer构建云主机base image时，执行以下两步，让期支持eth0的命名：
```
sudo sed -i '/^GRUB\_CMDLINE\_LINUX/s/\"$/\ net\.ifnames\=0\"/' /etc/default/grub

sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```




https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/enhanced-networking-ena.html#predictable-network-names-ena
