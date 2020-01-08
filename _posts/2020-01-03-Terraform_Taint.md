---
title: Terraform -- State and Taint
---



> Terraform可以查看state，当我们需要单独删除某个元素的时候，可以通过taint的方式单独删除某一个元素，可以用taint。



例如：

terraform state list列出所有的state

```
[root@6ca935d48faa SH81_MEDIA]# terraform state list
module.sh81_media_cluster.data.aws_route53_zone.pano_video
module.sh81_media_cluster.data.tencentcloud_image.pano_media_image
module.sh81_media_cluster.data.tencentcloud_key_pairs.sshkey
module.sh81_media_cluster.aws_route53_record.bastion[0]
module.sh81_media_cluster.aws_route53_record.kafka[0]
module.sh81_media_cluster.aws_route53_record.ms[0]
module.sh81_media_cluster.aws_route53_record.redis[0]
module.sh81_media_cluster.aws_route53_record.redis[1]
module.sh81_media_cluster.aws_route53_record.redis-vip
module.sh81_media_cluster.aws_route53_record.repo
module.sh81_media_cluster.aws_route53_record.rtms[0]
module.sh81_media_cluster.aws_route53_record.slb[0]
module.sh81_media_cluster.aws_route53_record.slb-vip
module.sh81_media_cluster.aws_route53_record.ts[0]
module.sh81_media_cluster.aws_route53_record.ts[1]
module.sh81_media_cluster.tencentcloud_cbs_storage.bastion_cbs[0]
module.sh81_media_cluster.tencentcloud_cbs_storage.kafka_cbs[0]
module.sh81_media_cluster.tencentcloud_cbs_storage.redis_cbs[0]
module.sh81_media_cluster.tencentcloud_cbs_storage.redis_cbs[1]
module.sh81_media_cluster.tencentcloud_cbs_storage_attachment.bastion_cbs_attachment[0]
module.sh81_media_cluster.tencentcloud_cbs_storage_attachment.kafka_cbs_attachment[0]
module.sh81_media_cluster.tencentcloud_cbs_storage_attachment.redis_cbs_attachment[0]
module.sh81_media_cluster.tencentcloud_cbs_storage_attachment.redis_cbs_attachment[1]
module.sh81_media_cluster.tencentcloud_clb_attachment.external_clb
module.sh81_media_cluster.tencentcloud_clb_attachment.external_clb_443
module.sh81_media_cluster.tencentcloud_clb_instance.external_clb
module.sh81_media_cluster.tencentcloud_clb_listener.external_tcp_443
module.sh81_media_cluster.tencentcloud_clb_listener.external_tcp_80
module.sh81_media_cluster.tencentcloud_ha_vip.redis
module.sh81_media_cluster.tencentcloud_instance.bastion[0]
module.sh81_media_cluster.tencentcloud_instance.kafka[0]
module.sh81_media_cluster.tencentcloud_instance.ms[0]
module.sh81_media_cluster.tencentcloud_instance.redis[0]
module.sh81_media_cluster.tencentcloud_instance.redis[1]
module.sh81_media_cluster.tencentcloud_instance.rtms[0]
module.sh81_media_cluster.tencentcloud_instance.slb[0]
module.sh81_media_cluster.tencentcloud_instance.ts[0]
module.sh81_media_cluster.tencentcloud_instance.ts[1]
module.sh81_media_cluster.tencentcloud_placement_group.bastion
module.sh81_media_cluster.tencentcloud_placement_group.bs
module.sh81_media_cluster.tencentcloud_placement_group.ds
module.sh81_media_cluster.tencentcloud_placement_group.kafka
module.sh81_media_cluster.tencentcloud_placement_group.ms
module.sh81_media_cluster.tencentcloud_placement_group.redis
module.sh81_media_cluster.tencentcloud_placement_group.rtms
module.sh81_media_cluster.tencentcloud_placement_group.slb
module.sh81_media_cluster.tencentcloud_placement_group.ts
module.sh81_media_cluster.tencentcloud_placement_group.turn
module.sh81_media_cluster.tencentcloud_route_table.pano_media_routetable1
module.sh81_media_cluster.tencentcloud_route_table_entry.vpn[0]
module.sh81_media_cluster.tencentcloud_security_group.bastion
module.sh81_media_cluster.tencentcloud_security_group.bs
module.sh81_media_cluster.tencentcloud_security_group.common
module.sh81_media_cluster.tencentcloud_security_group.ds
module.sh81_media_cluster.tencentcloud_security_group.kafka
module.sh81_media_cluster.tencentcloud_security_group.ms
module.sh81_media_cluster.tencentcloud_security_group.redis
module.sh81_media_cluster.tencentcloud_security_group.rtms
module.sh81_media_cluster.tencentcloud_security_group.slb
module.sh81_media_cluster.tencentcloud_security_group.ts
module.sh81_media_cluster.tencentcloud_security_group.turn
module.sh81_media_cluster.tencentcloud_security_group_lite_rule.bastion
module.sh81_media_cluster.tencentcloud_security_group_lite_rule.bs
module.sh81_media_cluster.tencentcloud_security_group_lite_rule.common
module.sh81_media_cluster.tencentcloud_security_group_lite_rule.ds
module.sh81_media_cluster.tencentcloud_security_group_lite_rule.kafka
module.sh81_media_cluster.tencentcloud_security_group_lite_rule.ms
module.sh81_media_cluster.tencentcloud_security_group_lite_rule.redis
module.sh81_media_cluster.tencentcloud_security_group_lite_rule.rtms
module.sh81_media_cluster.tencentcloud_security_group_lite_rule.slb
module.sh81_media_cluster.tencentcloud_security_group_lite_rule.ts
module.sh81_media_cluster.tencentcloud_security_group_lite_rule.turn
module.sh81_media_cluster.tencentcloud_subnet.pano_media_subnet1
module.sh81_media_cluster.tencentcloud_vpc.pano_media_vpc

[root@6ca935d48faa SH81_MEDIA]# terraform state show module.sh81_media_cluster.aws_route53_record.ts[1]
# module.sh81_media_cluster.aws_route53_record.ts[1]:
resource "aws_route53_record" "ts" {
    allow_overwrite = false
    fqdn            = "sh81ts002.bts.pano.video"
    id              = "Z34FROFM9IC0B2_sh81ts002.bts.pano.video_A"
    name            = "sh81ts002.bts.pano.video"
    records         = [
        "10.81.0.5",
    ]
    ttl             = 60
    type            = "A"
    zone_id         = "Z34FROFM9IC0B2"
}


```



将数据标示为taint：

```
[root@6ca935d48faa SH81_MEDIA]# terraform taint  module.sh81_media_cluster.aws_route53_record.ts[1]
Resource instance module.sh81_media_cluster.aws_route53_record.ts[1] has been marked as tainted.

[root@6ca935d48faa SH81_MEDIA]# terraform state show module.sh81_media_cluster.aws_route53_record.ts[1]
# module.sh81_media_cluster.aws_route53_record.ts[1]: (tainted)
resource "aws_route53_record" "ts" {
    allow_overwrite = false
    fqdn            = "sh81ts002.bts.pano.video"
    id              = "Z34FROFM9IC0B2_sh81ts002.bts.pano.video_A"
    name            = "sh81ts002.bts.pano.video"
    records         = [
        "10.81.0.5",
    ]
    ttl             = 60
    type            = "A"
    zone_id         = "Z34FROFM9IC0B2"
}
```



在将数据设置为taint之后，运行terraform apply就可以单独重建这个元素。