---
title: Terraform -- Environment & Module
---



> 本文记录目前对Terraform代码组织方式。



## Overall

目前terraform的代码结构如下：

```
[root@43036342e3a9 terraform]# tree
.
|-- environment
|   `-- tencent
|       |-- 0_DEV
|       |   `-- CENTRAL_BJ01
|       |       |-- main.tf
|       |       |-- provider.tf
|       |       |-- terraform.tfstate
|       |       `-- terraform.tfstate.backup
|       |-- 1_QA
|       |-- 2_BTS
|       `-- 3_PROD
`-- modules
    `-- tencent
        |-- centraldc
        |   |-- main.tf
        |   `-- variables.tf
        `-- mediadc

11 directories, 6 files
```

注：

在Modules定义了要部署完整的Cluster作为一个Module，其中定义了cluster中要部署的全部元素。

Environment仅输入需要的环境变量和provider参数。



## 为什么没有细化Module定义

出于以下考虑：

1. 目前我们的应用场景下，部署的cluster构架是固定的，因此定义在一个module并没有非常臃肿，后续的主机类型也不会膨胀到非常多，一个module目前看就可以cover。
2. 在特定应用场景下，很多东西都是固定的，过多的参数，反倒让使用者容易犯错。例如，instance type，挂盘大小，等等都是在Day 0固定的。
3. AWS，Ali，微软，Google已在Terraform官方提供抽象的module，后续如果要重构可以看Tencent有无计划提供。在Issue[1]中看看腾讯是否有后续计划。
4. 利用Environment管理不同环境的参数。



## Sample 

这是一个environment的例子：

```
 module "bj01_central_cluster" {
    source = "/code/terraform/modules/tencent/centraldc"
    dc_name = "bj01"
    az = "ap-beijing-5"
    vpc_cidr = "10.2.0.0/16"
    subnet1_cidr = "10.2.0.0/24"
    base_image = "pano-base-v0.3"
    gslb_count = 2
    gate_count = 2
    sirius_count = 2
    collapsar_count = 2
    dbstore_count = 2
    dbstore_volume_count = 2
    dbstore_volumesize = 10
    kafka_count = 3
    kafka_volume_count = 3
    kafka_volumesize = 10
    esm_count = 2
    esd_count = 3
    esd_volume_count = 3
    esd_volumesize = 10
    bastion_count = 1
    bastion_volume_count = 1
    bastion_volumesize = 10
}
```

可以看到基本仅需要定义不同VM类型的个数，VPC的IP使用范围，其他参数都在module中Day0固定。

当然这仅仅是我们目前使用场景比较固定，并不需要对其他进行变更。terraform.tfstate是重要信息，可以考虑放在本地+远程KV存贮。





经测试启动50台机器的环境大概需要2分钟，可以满足我们目前的需求，目前腾讯云对于每个AZ中能够启动的按需分配的VM数目限制为30或者60（不同Region可能不同）。



## Refer

[1] https://github.com/terraform-providers/terraform-provider-tencentcloud/issues/193
