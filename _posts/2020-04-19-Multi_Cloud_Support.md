---
title: DevOps -- MultiCloud Support 
---

我们最初只支持腾讯云部署，最近在支持阿里云。以下内容需要在支持多云时有一定工作：
* Terraform支持新Cloud Provider，并对接到自己的Admin管理系统。
* 为了节约流量节省成本，每个云都就近使用对象存储。特别时录制文件，因此和录制文件相关的服务需要支持多provider。
* 由于有海外加速，因此多云支持时，需要采购专线，并且支持专线接入。
* 就近支持录制文件CDN下载。
* 支持HAVIP（Active - Standby）

