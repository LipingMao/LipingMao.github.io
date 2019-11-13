---
title: Terraform -- Resource
---



> 本文介绍Terraform的Resource的概念。阅读本文前建议先阅读官方Terraform -- Intro中的腾讯云系列文章对Terraform有基本了解。



先看一个例子：

```shell
resource "aws_instance" "example" {
  ami           = "ami-a1b2c3d4"
  instance_type = "t2.micro"
  count　        = 5

  iam_instance_profile = aws_iam_instance_profile.example
  
  lifecycle {
    create_before_destroy = true
  }

  depends_on = [
    aws_iam_role_policy.example,
  ]
}
```



一个Resource的定义是： resource "aws_instance" "example" { }

resource定义了这是一个resource block。 

"aws_instance"是一种Resource Type，可以在[2]找到Terraform支持的Resource Type。

"example" 为Reource的名称。



所有在{}内定义的是metadata，metadata分成两类，

一类是Resource Type中定义的每个resource特定的metadata，如例子中ami是aws_instance这个资源类型特有的。

第二类是所有Type都支持的通用类型，如count, lifecycle, depends_on 等等。



当Terraform创建资源后，默认会将状态保存在terraform.tfstate文件中，当然你也可以配置远端存储，将其保存在远端，Terraform目前支持Consul, etcd 等，具体可参见[3]。 你可以通过Terraform state命令查看状态相关信息。如下例所示：

```shell
## 列出所有的state

[root@43036342e3a9 qa-tencent-bj]# terraform state list
data.tencentcloud_image.pano_image
tencentcloud_clb_attachment.internal_clb[0]
tencentcloud_clb_attachment.internal_clb[1]
tencentcloud_clb_instance.internal_clb
tencentcloud_clb_listener.internal_tcp_80
tencentcloud_instance.gate[0]
tencentcloud_instance.gate[1]
tencentcloud_key_pair.sshkey
tencentcloud_placement_group.gate
tencentcloud_security_group.common
tencentcloud_security_group.gate
tencentcloud_security_group_lite_rule.common
tencentcloud_security_group_lite_rule.gate
tencentcloud_subnet.pano_subnet1
tencentcloud_vpc.pano_vpc


## 查看某个state

[root@43036342e3a9 qa-tencent-bj]# terraform state show tencentcloud_clb_instance.internal_clb
# tencentcloud_clb_instance.internal_clb:
resource "tencentcloud_clb_instance" "internal_clb" {
    clb_name                  = "internal_bj01_clb"
    clb_vips                  = [
        "10.100.0.9",
    ]
    id                        = "lb-2jror54d"
    network_type              = "INTERNAL"
    project_id                = 0
    subnet_id                 = "subnet-pa5w92fr"
    target_region_info_region = "ap-beijing"
    target_region_info_vpc_id = "vpc-b5zlofj8"
    vpc_id                    = "vpc-b5zlofj8"
}
```


以下metadata是每个resource都支持的通用字段：

* depends_on, for specifying hidden dependencies
* count, for creating multiple resource instances according to a count
* for_each, to create multiple instances according to a map, or set of strings
* provider, for selecting a non-default provider configuration
* lifecycle, for lifecycle customizations
* provisioner and connection, for taking extra actions after resource creation


其中：
depends_on可以显式指定Resource依赖关系。
count/for_each可以指定同个Resource多个副本，例如创建5台相同类型的VM。
lifecycle可以修改默认的lifecycle动作，例如默认情况下Terraform会先删除，再创建，利用此配置可以先创建后删除。此字段同样可以控制比如忽略某些字段的变更，例如可以忽略image的变化。



[1] https://www.terraform.io/docs/configuration/resources.html

[2] https://www.terraform.io/docs/providers/index.html

[3] https://www.terraform.io/docs/backends/types/index.html
