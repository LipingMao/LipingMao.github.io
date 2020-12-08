---
title: DevOps -- stop firewalld cause cpu high
---

调用stop firewalld时，会导致rm_mod nf_contrack 卸载，内核问题会导致一个cpu sys load一直为100%， rmmod nf_contrack会执行失败


[1] https://bugzilla.redhat.com/show_bug.cgi?id=1397274
