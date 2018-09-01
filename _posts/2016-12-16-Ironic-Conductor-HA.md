---
title: OpenStack -- Ironic-Conductor HA
---

## Background

> We need to deploy Ironic for private openstack cloud. Ironic-Conductor need to be deployed
in HA Mode. We are using OSP9(Mitaka Release), in this blog, I will paste the problem we hit.
And how Ironic-Conductor HA works.

## Deploy Arch for Ironic-Conductor

The HA deploy Arch is quite simple for ironic-conductor, we have 3 controller nodes.
Ironic-Conductor will be installed in each of the nodes.

## How it works

Ironic-Conductor is a little bit different than other stateless service in openstack.
It will provision pxe configuration(in the case, baremetal server need use pxe when reboot).
So it is non-stateless. When you boot a Baremetal, ironic will choose one of the Ironic-Conductor
to handle the work. And it will be recorded into DB, the node will have conductor_affinity id to
record which ironic-conductor it is bond with. And ironic-conductor will use Consistent Hashing to
caculate which ironic-conductor the node should use. when ironic-conductor is down, by default, ironic-conductor
will recaculate the Consistent Hashing every 180s. Then, other alived ironic-conductor will take over
existed node which bond with the failed node.

## What's the problem

In Mitaka, currently we get the following bug(one of them code has not been merged), they can work after these bugs fixed.
[1]  https://review.openstack.org/#/c/246036/
[2]  https://review.openstack.org/#/c/246033/
