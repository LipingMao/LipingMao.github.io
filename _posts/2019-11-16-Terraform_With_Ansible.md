---
title: Terraform -- Integration with Ansible
---



> 本文记录目前对Terraform与Ansible进行交互。



## Inventory

Terraform生成基础设施后，Ansible要对应用进行配置管理，因此Ansible本身的Inventory信息就需要来源于Terraform部署生成的环境。

自然的，Ansible就需要使用Dynamic Inventory的方式从Terraform的state文件中分析出，例如： VM的hostname， IP 地址， Loadbalance的IP等信息。



## Data Sources

新版本的Terraform支持Data Source功能，可以读取过滤需要的信息并导出到外部json文件中，用于外部系统分析 ，见 [Data Source](https://www.terraform.io/docs/configuration/data-sources.html) 。 以下为腾讯云为例，导出所有VM，CLB的信息：

```
# output data cource for ansible
data "tencentcloud_instances" "vms" {
  vpc_id = "${tencentcloud_vpc.pano_vpc.id}"
  result_output_file = "vms.json"
}

data "tencentcloud_clb_instances" "external" {
  clb_id             = "${tencentcloud_clb_instance.external_clb.id}"
  result_output_file = "external.json"
}

data "tencentcloud_clb_instances" "internal" {
  clb_id             = "${tencentcloud_clb_instance.internal_clb.id}"
  result_output_file = "internal.json"
}
```



相关信息可以被保存在json文件中, 已CLB为例：

```
[root@43036342e3a9 output]# cat external.json
[
	{
		"clb_id": "lb-nlix74ap",
		"clb_name": "external_bj01_clb",
		"clb_vips": [
			"140.143.116.27"
		],
		"create_time": "2019-11-16 19:49:15",
		"network_type": "OPEN",
		"project_id": 0,
		"security_groups": [],
		"status": 1,
		"status_time": "2019-11-16 19:49:51",
		"subnet_id": "",
		"tags": {},
		"target_region_info_region": "ap-beijing",
		"target_region_info_vpc_id": "vpc-d0sov4pu",
		"vpc_id": "vpc-d0sov4pu"
	}
]
```

可以通过解析JSON获取需要的Inventory信息为Ansible使用。



还有一种办法是使用第三方写的[Ansible Provider](https://github.com/nbering/terraform-inventory) ,通过在module中定义Ansible Host/Group等信息保存在Terraform State中，然后通过python脚本分析state中特定的结构解析出Inventory，此方法更为通用。



这两个方法都可以达到目的，第二种方法更通用，不过需要引入一个第三方Provider。在结构并不复杂的场景下，第一种方式看起来更加直接一些。可以根据需求选取适合自己的方式。

