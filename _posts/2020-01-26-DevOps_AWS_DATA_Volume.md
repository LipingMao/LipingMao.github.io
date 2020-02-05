---
title: DevOps -- AWS Data Volume 
---



## 如何为AWS的VM添加第二块Data盘：

首先在AWS console中创建volume并挂载到对应vm。然后进入系统，以CentOS7为例。

查看fdisk，可以看到/dev/nvme1n1是新盘：

```
[root@nx01test004 data]# fdisk -l

Disk /dev/nvme0n1: 21.5 GB, 21474836480 bytes, 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000b956b

Device Boot Start End Blocks Id System
/dev/nvme0n1p1 * 2048 41943006 20970479+ 83 Linux

Disk /dev/nvme1n1: 10.7 GB, 10737418240 bytes, 20971520 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```



查看盘上文件类型，看到是data，没有初始化文件系统：

```
[root@nx01test004 data]# file -s /dev/nvme1n1
/dev/nvme1n1: data
```



初始化xfs文件系统：

```
[root@nx01test004 data]# mkfs -t xfs /dev/nvme1n1
meta-data=/dev/nvme1n1 isize=512 agcount=4, agsize=655360 blks
= sectsz=512 attr=2, projid32bit=1
= crc=1 finobt=0, sparse=0
data = bsize=4096 blocks=2621440, imaxpct=25
= sunit=0 swidth=0 blks
naming =version 2 bsize=4096 ascii-ci=0 ftype=1
log =internal log bsize=4096 blocks=2560, version=2
= sectsz=512 sunit=0 blks, lazy-count=1
realtime =none extsz=4096 blocks=0, rtextents=0

[root@nx01test004 /]# file -s /dev/nvme1n1
/dev/nvme1n1: SGI XFS filesystem data (blksz 4096, inosz 512, v2 dirs)
```



mount磁盘到目录：

```
[root@nx01test004 /]# mount /dev/nvme1n1 /data
```



在/etc/fstab中添加以下内容，确保reboot后恢复：

```
/dev/nvme1n1 /data xfs defaults 0 0
```

## 