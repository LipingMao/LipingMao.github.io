---
title: Terraform -- Misc
---



> 本文记录Terraform一些拾遗。



## Provider



先看一个例子：

```
provider "tencentcloud" {
  version    = "~> 1.22.0"
  secret_id  = "ID"
  secret_key = "KEY"
  region     = "ap-beijing"
}
```



此例中指定了provider是腾讯云，由于tencentcloud是terraform官方支持的，在运行terraform init后，会自动下载腾讯云module。本例中指定安装版本是大于1.22.0 。



可以给Provider指定别名，如下所示：

```
# The default provider configuration
provider "aws" {
  region = "us-east-1"
}

# Additional provider configuration for west coast region
provider "aws" {
  alias  = "west"
  region = "us-west-2"
}
```



## Input

可以定义变量作为：

```
variable "image_id" {
  type        = string
  default     = "abc"
  description = "The id of the machine image (AMI) to use for the server."
}
```

支持的type有：

 ```
string
number
bool
 ```

支持的collection有：

 ```
list(<TYPE>)
set(<TYPE>)
map(<TYPE>)
object({<ATTR NAME> = <TYPE>, ... })
tuple([<TYPE>, ...])
 ```

在module中可以通过引用var.image_id访问此变量。



## Output

类似的，可以定义output变量方便其他地方引用，如下例子：

```
output "db_password" {
  value       = aws_db_instance.db.password
  description = "The password for logging in to the database."
  sensitive   = true
}
```

注：如果是敏感信息可以定义sensitive为true，不明文显示。



## Module

为了模块化terraform的配置，可以将功能内聚的一些配置组成一个模块，例如一个虚拟机和它的securitygroup，硬盘。可以通过source指定使用什么模块：

 ```
module "servers" {
  source = "./app-cluster"

  servers = 5
}
 ```

可以通过 “module.模块名.output变量” 访问模块的输出变量：

 ```
resource "aws_elb" "example" {
  # ...

  instances = module.servers.instance_ids
}
 ```



## Data Sources

不同的provider支持Data Sources， 以AWS为例：

```
data "aws_ami" "example" {
  most_recent = true

  owners = ["self"]
  tags = {
    Name   = "app-server"
    Tested = "true"
  }
}
```



## Version

0.12 是一个major release，Terraform DSL在这个版本发生了变更，如果从现在开始使用，请使用0.12之后版本。



## Refer

[1] https://www.terraform.io/docs/configuration/index.html