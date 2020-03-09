---
title: DevOps -- Extend Data Volume on Tencent Cloud
---

> 需要在运行时给虚拟机增加数据盘容量，希望尽可能减少对虚拟机的影响。

按以下步骤扩容：
*  修改Terraform的配置，将磁盘扩展到合适容量，并运行terraform apply。
*  在OS内部进行扩容：
a） 查看文件系统类型：
```
[root@hk80bastion001 ~]# parted -l
Warning: Unable to open /dev/sr0 read-write (Read-only file system).  /dev/sr0
has been opened read-only.
Error: /dev/sr0: unrecognised disk label
Model: QEMU QEMU DVD-ROM (scsi)
Disk /dev/sr0: 43.0MB
Sector size (logical/physical): 2048B/2048B
Partition Table: unknown
Disk Flags:
 
Model: Virtio Block Device (virtblk)
Disk /dev/vda: 53.7GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:
 
Number  Start   End     Size    Type     File system  Flags
 1      1049kB  53.7GB  53.7GB  primary  ext4         boot
 
 
Model: Virtio Block Device (virtblk)
Disk /dev/vdb: 21.5GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:
 
Number  Start   End     Size    Type     File system  Flags
 1      1049kB  21.5GB  21.5GB  primary  ext4
```
b) 扩展磁盘容量：
growpart /dev/vdb 1

c）根据不同文件系统，进行扩容：
如果是ext4， resize2fs /dev/vdb1
如果是xfs， xfs_growfs /dev/vdb1

*  查看并确认扩容成功：
```
[root@hk80bastion001 ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        3.9G     0  3.9G   0% /dev
tmpfs           3.9G   24K  3.9G   1% /dev/shm
tmpfs           3.9G  1.8M  3.9G   1% /run
tmpfs           3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/vda1        50G  8.7G   39G  19% /
/dev/vdb1        20G  8.3G   11G  45% /data
```

PS:
腾讯云的OS盘是不允许扩容的，固定的50G，只能重建镜像。
