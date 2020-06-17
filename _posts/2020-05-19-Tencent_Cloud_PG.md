---
title: DevOps -- Tencent Cloud Placement Group
---

腾讯云支持变更CVM所属的placement group，但是不支持将CVM从placement group里面取出。

所以在不影响现在已经部署的CVM的前提下，需要做如下的处理。
1. tencentcloud_instance的lifecycle的ignore_changes加上placement_group_id
2. 从tfstate里面删除所有的tencentcloud_placement_group

