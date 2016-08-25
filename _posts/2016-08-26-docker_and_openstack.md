---
title: Docker and Openstack
---
## Backgroup
> There is so many ways and choices to let docker work with openstack. Here is a quick summary how they can work together.

## How to work together?

### Nova-Docker Driver

Use Docker as "Hyperviser" just like libvirt(kvm), Xen.

[Nova-Docker Driver Code Base](https://github.com/openstack/nova-docker/tree/master)
[Nova Docker Driver Blog](https://sreeninet.wordpress.com/2015/06/14/openstack-and-docker-part-1/)

### Heat

Use Heat to orchestrate docker container on Openstack.

[Heat and Docker Blog](https://sreeninet.wordpress.com/2015/06/14/openstack-and-docker-part-2/)

### Magnum
K8S, Mesos, Swarm cluster as a service on Openstack.

[Magnum Wiki](https://wiki.openstack.org/wiki/Magnum)

### Kolla / Kolla-k8s
Use Docker to containize openstack service, then use anisble / k8s to orchestrate the deploy of Openstack.

[Kolla & Kolla-k8s](http://docs.openstack.org/developer/kolla-kubernetes/quickstart.html)

### Kuryr / Kuryr-libnetwork / Kuryr-k8s / FuXi

Docker libnetwork remote Driver / Volume Driver to use neutron / Cinder as backend for docker.

[Kuryr Repo](https://github.com/openstack?utf8=%E2%9C%93&query=kuryr)
