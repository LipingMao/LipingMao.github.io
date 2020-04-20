---
title: DevOps -- Ansible serial and order
---

当Ansible同时使用使用serial和order时，order只能控制同一个serial batch中的顺序。如果serial是1的话，则order永远都是无序的，因为serial本身是从inventory中无序选取主机。
