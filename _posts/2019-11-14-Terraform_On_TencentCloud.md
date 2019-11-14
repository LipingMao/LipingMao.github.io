---
title: Terraform -- Practice on Tencent Cloud
---



> 本文记录在使用Terraform在TencentCloud上的一些实践。



## Environment

假定应用场景如下，在腾讯云上构建以下环境：

```
1. 构建VPC， Subnet。
2. 管理SSH Key。
3. 管理多台CVM，并带数据盘。
4. CVM使用Placement Group确保在不同的HOST上。
5. 定义Common Security Group，以及Service级别Security Group。
6. 定义CLB，listener，将CLB为CVM提供TCP Layer4负载均衡。
```



## Play with Terraform

配置腾讯云Provider：

```
provider "tencentcloud" {
  version    = "~> 1.22.0"
  secret_id  = "$YOUR_ID"
  secret_key = "$YOUR_KEY"
  region     = "ap-beijing"
}
```

此例中，指定了目前最新的provider版本，指定Region是北京。



创建VPC，Subnet的例子：

```
# VPC
resource "tencentcloud_vpc" "pano_vpc" {
  name       = "pano_vpc_${var.dc_name}"
  cidr_block = "${var.vpc_cidr}"
}

# Subnet
resource "tencentcloud_subnet" "pano_subnet1" {
  availability_zone = "${var.az}"
  name              = "pano_subnet1_${var.dc_name}"
  vpc_id            = "${tencentcloud_vpc.pano_vpc.id}"
  cidr_block        = "${var.subnet1_cidr}"
}
```

其中var.XXX为定义的变量，可以根据不同环境输入, 例如：

```
## Common
variable "dc_name" {
  type        = string
  description = "DC Name which will be deployed"
  default     = "bj01"
}


variable "az" {
  type        = string
  description = "AZ Name which will be deployed"
  default     = "ap-beijing-5"
}


variable "vpc_cidr" {
  type        = string
  description = "VPC CIDR"
  default     = "10.100.0.0/16"
}


variable "subnet1_cidr" {
  type        = string
  description = "Subnet CIDR"
  default     = "10.100.0.0/24"
}
```



通过datasource 获取指定的Image，后续启动虚拟机时需要使用image id：

```
data "tencentcloud_image" "pano_image" {
  image_name_regex = "${var.base_image}"
}
```

其中变量的定义为：

```
variable "base_image" {
  type        = string
  description = "Base Image"
  default     = "pano-base-v0.3"
}
```



定义sshkey，后续启动虚拟机使用key登陆：

```
resource "tencentcloud_key_pair" "sshkey" {
  key_name   = "pano_ssh_key"
  public_key = "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCLGAgrXY7G3HwO8voNKokyiAaUFtLgrSi6SuIfuvySddhTrCa0/oN+ro88j+pW8LS2zUWz12d8tqp/0Qmd8jhNVf6ITZwrgc+9gTX+sQWrrpx5jGGnrv9ISDD3NCMJVhJ514VWfH6sa5YKmYIQ49/DS/Iak88AmVbQAxKZHShvFlBS/xZ4QEkQy1731QmhK4t1As/ICfla2Qiv/rI7VxWPT21kVT4dH2Y1coqCmXymRgtU8zj4nVRi3XjgVSYIbqZFLH7aBjkeIE0Wpg8WOY7zZNOcPcGYZk/vIqyZdPUTy3S/jBcUJLqYUZvxeJYK+sKeAEFN+q5PnwGQ54nTOLaj pano"
  lifecycle {
    ignore_changes = [
      public_key,
    ]
  }
}
```

注：

此处定义了忽略public_key的变化，否则每次都会replace sshkey。



以下定义了内部CLB，外部CLB：

```
## clb
resource "tencentcloud_clb_instance" "internal_clb" {
  network_type = "INTERNAL"
  clb_name     = "internal_${var.dc_name}_clb"
  project_id   = 0
  vpc_id       = "${tencentcloud_vpc.pano_vpc.id}"
  subnet_id    = "${tencentcloud_subnet.pano_subnet1.id}"
}

resource "tencentcloud_clb_listener" "internal_tcp_80" {
  clb_id                     = "${tencentcloud_clb_instance.internal_clb.id}"
  listener_name              = "internal_${var.dc_name}_tcp_80"
  port                       = 80
  protocol                   = "TCP"
  health_check_switch        = true
  health_check_time_out      = 2
  health_check_interval_time = 5
  health_check_health_num    = 3
  health_check_unhealth_num  = 3
  session_expire_time        = 30
  scheduler                  = "WRR"
}

resource "tencentcloud_clb_attachment" "internal_clb" {
  clb_id      = "${tencentcloud_clb_instance.internal_clb.id}"
  listener_id = "${tencentcloud_clb_listener.internal_tcp_80.id}"

  dynamic "targets" {
    for_each = [for value in tencentcloud_instance.gate: value.id]
    content {
      instance_id = targets.value
      port        = 81
      weight      = 10
    }
  }
}


resource "tencentcloud_clb_instance" "external_clb" {
  network_type = "OPEN"
  clb_name     = "external_${var.dc_name}_clb"
  project_id   = 0
  vpc_id       = "${tencentcloud_vpc.pano_vpc.id}"
}

resource "tencentcloud_clb_listener" "external_tcp_80" {
  clb_id                     = "${tencentcloud_clb_instance.external_clb.id}"
  listener_name              = "external_${var.dc_name}_tcp_80"
  port                       = 80
  protocol                   = "TCP"
  health_check_switch        = true
  health_check_time_out      = 2
  health_check_interval_time = 5
  health_check_health_num    = 3
  health_check_unhealth_num  = 3
  session_expire_time        = 30
  scheduler                  = "WRR"
}

resource "tencentcloud_clb_attachment" "external_clb" {
  clb_id      = "${tencentcloud_clb_instance.external_clb.id}"
  listener_id = "${tencentcloud_clb_listener.external_tcp_80.id}"

  dynamic "targets" {
    for_each = [for value in tencentcloud_instance.gate: value.id]
    content {
      instance_id = targets.value
      port        = 80
      weight      = 10
    }
  }
}

```



定义Placement policy防止VM schedule到同一个Host：

```
## placement policy
resource "tencentcloud_placement_group" "gate" {
  name = "${var.dc_name}_gate"
  type = "HOST"
}
```



定义安全策略组：

```
## security group
resource "tencentcloud_security_group" "gate" {
  name        = "${var.dc_name}_gate"
  description = "security group for gate"
}

resource "tencentcloud_security_group_lite_rule" "gate" {
  security_group_id = "${tencentcloud_security_group.gate.id}"

  ingress = [
    "ACCEPT#0.0.0.0/0#80#TCP",
    "ACCEPT#0.0.0.0/0#443#TCP",
    "ACCEPT#10.0.0.0/8#ALL#ALL",
  ]

  egress = [
    "ACCEPT#0.0.0.0/0#ALL#ALL",
  ]
}
```



定义虚拟机：

```
## server
resource "tencentcloud_instance" "gate" {
  instance_name              = "${var.dc_name}${var.gate_prefix}${format("%03d", count.index + 1)}"
  availability_zone          = "${var.az}"
  image_id                   = "${data.tencentcloud_image.pano_image.image_id}"
  allocate_public_ip         = true
  key_name                   = "${tencentcloud_key_pair.sshkey.id}"
  security_groups            = ["${tencentcloud_security_group.common.id}", "${tencentcloud_security_group.gate.id}"]
  placement_group_id         = "${tencentcloud_placement_group.gate.id}"
  instance_type              = "${var.gate_instance_type}"
  system_disk_type           = "CLOUD_PREMIUM"
  system_disk_size           = 50
  hostname                   = "${var.dc_name}${var.gate_prefix}${format("%03d", count.index + 1)}"
  project_id                 = 0
  vpc_id                     = "${tencentcloud_vpc.pano_vpc.id}"
  subnet_id                  = "${tencentcloud_subnet.pano_subnet1.id}"
  internet_max_bandwidth_out = 100
  count                      = "${var.gate_count}"
}
```





## Tips for Tencent Provider

* 使用Terraform 0.12以上版本，如果你此时开始的话，0.12版本用了新的Terraform DSL，支持了更多新特性。腾讯云provider和0.12已经完全兼容。
* 腾讯云大概是从今年开始陆续支持Terraform，目前社区比较积极[7]。
* 主要的功能没有问题，不过使用测试过程中还是有些小bug，如[2] [8]。不过响应还是比较积极的。
* 有些功能并不完善，如不支持HAVIP（预计12月支持）[3]，不支持DNS（预计明年2月支持）[4]
* 目前腾讯云的计费模式，只支持包月或者按需收费[5]，如果要使用terraform经常性对VM进行rotate，费用只能按照按需收费进行，比较可惜，腾讯云目前没有明确timeline什么时候可以支持类似AWS RI灵活计费和阿里云的RI[6]. 这种情况下，非常快速的更换虚拟机就会产生费用问题（开发/升级运维场景下需要）。目前可以使用Terraform作为IaC工具。



## Refer

[1] https://www.terraform.io/docs/configuration/index.html

[2] https://github.com/terraform-providers/terraform-provider-tencentcloud/issues/185

[3] https://github.com/terraform-providers/terraform-provider-tencentcloud/issues/184

[4] https://github.com/terraform-providers/terraform-provider-tencentcloud/issues/186

[5] https://github.com/terraform-providers/terraform-provider-tencentcloud/issues/192

[6] https://www.jianshu.com/p/46a5d4b14311

[7] https://github.com/terraform-providers/terraform-provider-tencentcloud/graphs/contributors

[8] https://github.com/terraform-providers/terraform-provider-tencentcloud/issues/194