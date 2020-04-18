---
title: DevOps -- Ansible Inventory Sorted Host
---

> 有时需要Ansible中的节点运行是有序的，例如，进行滚动升级时希望每次都是按照特定顺序执行。Ansible中的Group使用的Python字典，默认是无序的。

可以在host中添加以下内容，转成有序：
```
order: sorted
```
