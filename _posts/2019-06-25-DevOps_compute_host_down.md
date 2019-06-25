---
title: DevOps -- compute node host down
---

## Background

> We notice every week we have about 1-2 physical compute nodes will be down, we have about 1500 physcial servers in Production which runs as Compute nodes, there is no error log in messages around the reboot/hang. We need to figure out the root cause.


## Debug the root cause

We are using UCS C220 M4 servers for most of the compute nodes, we find the following issues :
### 1. Firmware bug

Need to upgrade firmware to 3.0(3f) to avoid the following bug in firmware:
https://bst.cloudapps.cisco.com/bugsearch/bug/CSCve95795/?rfs=iqvred

### 2. CPU c0/c1 state

Firmware has bug when CPU state set to other than c0/c1, there is possibility to have host down. But even you set up c0/c1 in BIOS, CentOS by default will ignore this setting, you could set up force_latency=1 in tuned profile.
https://bst.cloudapps.cisco.com/bugsearch/bug/CSCvd86049
https://www.golinuxhub.com/2018/06/what-cpu-c-states-check-cpu-core-linux.html

### 3. EDAC bug in kernel

Our CentOS kernel get the following bug will cause the crash.
https://access.redhat.com/solutions/3160241


## Summary

It is painful to debug a issue which is not able to reproduce in lab and happen randomly. We analysis the issue by check coredump, Cisco UCS bug list, search RHEL KB. Basically for host down randomly or crash, BIOS / Kernel is somewhere worth to pay more attention to.
