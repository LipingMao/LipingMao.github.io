---
title: Terraform -- AWS Route53
---



> 腾讯云目前Terraform暂时不支持DNS设置，我们目前DNS使用的是AWS提供的Route53，本文大致介绍了如何使用Terraform和Route53为腾讯云的VM及CLB添加DNS的操作。



首先配置AWS的provider，如下例：

```
provider "tencentcloud" {
  version    = "~> 1.25.0"
  secret_id  = "XXX"
  secret_key = "YYY"
  region     = "ap-beijing"
}

provider "aws" {
  access_key = "ZZZ"
  secret_key = "NNN"
  region = "us-west-1"
}
```



一般来说都是实现创建好的zone，此例中我们使用"pano.video.":

```
data "aws_route53_zone" "pano_video" {
  name = "pano.video."
}
```



以下实例分别配置了VIP的A记录，以及不同服务的CNAME：

```
# Internal LB
resource "aws_route53_record" "internal-vip" {
  allow_overwrite = false
  name            = "${var.dc_name}-internal-api.${var.domain}"
  ttl             = 60
  type            = "A"
  zone_id         = "${data.aws_route53_zone.pano_video.zone_id}"
  records         = ["${tencentcloud_clb_instance.internal_clb.clb_vips[0]}"]
}

# Kibana
resource "aws_route53_record" "internal-kibana-vip" {
  allow_overwrite = false
  name            = "${var.dc_name}-kibana.${var.domain}"
  ttl             = 60
  type            = "CNAME"
  zone_id         = "${data.aws_route53_zone.pano_video.zone_id}"
  records         = ["${var.dc_name}-internal-api.${var.domain}"]
}
```



为虚拟机配置HOSTNAME也很简单，用count随着实例数目配置相应DNS：

```
resource "aws_route53_record" "gate" {
  allow_overwrite = false
  name            = "${tencentcloud_instance.gate[count.index].instance_name}.${var.domain}"
  ttl             = 60
  type            = "A"
  zone_id         = "${data.aws_route53_zone.pano_video.zone_id}"
  records         = ["${tencentcloud_instance.gate[count.index].private_ip}"]
  count           = "${var.gate_count}"
}
```

