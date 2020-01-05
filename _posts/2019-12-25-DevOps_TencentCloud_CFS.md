---
title: DevOps -- Tencent Cloud CFS
---



> 本文记录了腾讯云的CFS使用及基本测试。



### 业务场景

我们的业务有生成Cloud Recording需求，云录制本身产生后会被存放到对象存储中，但是云录制文件的生成是一个异步的过程，dump server需要先将录制的裸数据存放在本地，转码服务会对裸数据进行转码，因此在上传到对象存储之前，就有需求本地存放未转码的数据，以及未及时上传到对象存储的数据。

虚拟机本地数据的存放在云上无非两个选择：

1. 本地配置第二块块存储存放额外数据。
2. 使用云上提供的文件存储服务（腾讯云叫CFS）。



使用第一种方式时，对业务系统的就要求，可“预期”本地会产生的临时数据量，并且，由于转码服务本身是分布式的，那每一台转码服务器本身就需要一定量的额外磁盘。因此对于Day0容量规划有一定要求。而dump server存放dump数据由于是实时的，即如果当时不能dump下来，那这些录制数据就会丢失，因此在考虑Day0的capacity时，一定会多留余量。本地的额外盘只能按照业务高峰进行容量评估。这样在规划容量时，不免有些浪费。

第二种方式是利用云上的文件存储服务，即类似于NFS的方式，每台业务服务器挂在到相同NFS存储目录中，所有机器share相同的目录，就仅需要对一个目录进行总容量规划。而腾讯云的CFS是按需付费动态扩展容量大小，因此就不会有容量规划问题。



### 什么是CFS



这是腾讯云官方解释：

> 文件存储（Cloud File Storage，CFS）提供了可扩展的共享文件存储服务，可与腾讯云的 CVM 、容器、批量计算等服务搭配使用。CFS 提供了标准的 NFS 及 CIFS/SMB 文件系统访问协议，为多个 CVM 实例或其他计算服务提供共享的数据源，支持弹性容量和性能的扩展，现有应用无需修改即可挂载使用，是一种高可用、高可靠的分布式文件系统，适合于大数据分析、媒体处理和内容管理等场景。
>
> 文件存储接入简单，您无需调节自身业务结构，或者是进行复杂的配置。只需三步即可完成文件系统的接入和使用：创建文件系统，启动服务器上文件系统客户端，挂载创建的文件系统。[1]



说人话就是，不用关心大小的nfs，你用了多少按小时最大容量收费，费用和块存储的单价相同。



有几个问题稍微值得注意一下：

> 是否能跨AZ？Region？

由于NFS mount时仅需要网络可达，因此只要是网络可达，就可以访问，但是跨地域时有时延问题，一般不会垮地域。多AZ可以使用同一个nfs，不过nfs本身属于一个AZ，因此本质上如果跨AZ访问，会丧失一些可靠性。（NFS所在AZ出问题的话，影响到另一个AZ）。



> NFS访问控制？

提供了类似ACL的功能，控制什么样的IP可以挂载这个NFS。这和普通NFS类似。



> 自动化怎么搞？

Terraform Tencent Provider已经支持CFS相关操作[2]。



### 使用及测试

CFS的初次体验如下：

```
#1. Console中创建文件存储，默认情况下所有人都能访问。

#2. 安装nfs客户端，及挂载mount点
sudo yum install nfs-utils
midir /nfstest
mount -t nfs -o vers=4  10.1.0.27:/ /nfstest

#3. 查看nfs点，可以看到默认情况下分配了10G磁盘
[root@sh01gslb001 nfstest]# df -h
Filesystem      Size  Used Avail Use% Mounted on
...
10.1.0.27:/      10G  32M  10G  1% /nfstest

#4. 测试大文件（生成了10G）及自动扩展，发现扩展到了20G
[root@sh01gslb002 nfstest]# df -h
Filesystem      Size  Used Avail Use% Mounted on
...
10.1.0.27:/      20G   32M   20G   1% /nfstest

```





### Refer

[1] https://cloud.tencent.com/document/product/582/9127

[2] https://www.terraform.io/docs/providers/tencentcloud/d/cfs_access_rules.html