---
title: OpenStack -- rbac
---
## Backgroup

> Neutron has rbac to network and qos policy in Mitaka, this blog will have a brief introduce to this feature.

## What's the problem before RBAC?

> Before RBAC, we can only share network to all tenants or make network dedicate to one tenant. This would be problem, if we want to share network within some tenants.

## How to do RBAC?

In Mitaka, RBAC support both network and qos policy, in this blog we will use network as an example. For network, mitaka support to set rbac on both private and external network. I will use external network as an example here. First of all, we create an external network:

```
# neutron net-create limao_test --router:external true
Created a new network:
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | True                                 |
| availability_zone_hints   |                                      |
| availability_zones        |                                      |
| created_at                | 2016-09-28T02:39:26                  |
| description               |                                      |
| id                        | 604970a6-7970-47a5-90b2-b5822757cc0c |
| ipv4_address_scope        |                                      |
| ipv6_address_scope        |                                      |
| is_default                | False                                |
| mtu                       | 1450                                 |
| name                      | limao_test                           |
| port_security_enabled     | True                                 |
| provider:network_type     | vxlan                                |
| provider:physical_network |                                      |
| provider:segmentation_id  | 60022                                |
| qos_policy_id             |                                      |
| router:external           | True                                 |
| shared                    | False                                |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| tags                      |                                      |
| tenant_id                 | fa7c66615e8948039ec06f2bf7c13be3     |
| updated_at                | 2016-09-28T02:39:26                  |
+---------------------------+--------------------------------------+
```

Check rbac, you will find a new rbac rule created for this network, and by default, it will allow all tenants to see this external network:

```
[root@ci91hf3ctr001 ~]# neutron rbac-list --object_id=604970a6-7970-47a5-90b2-b5822757cc0c -c id -c target_tenant
+--------------------------------------+---------------+
| id                                   | target_tenant |
+--------------------------------------+---------------+
| 5d7cd511-a129-44ce-90bb-bf729caa1182 | *             |
+--------------------------------------+---------------+
```

RBAC update to the tenant which you want let it access.

```
[root@ci91hf3ctr001 ~]# neutron rbac-update --target-tenant $TENANT_ID 5d7cd511-a129-44ce-90bb-bf729caa1182
```
