---
title: Neutron Memo(1) -- Port Security
---
## Backgroup

> Neutron has security group / anti-ip-spoofing / anti-arp-spoofing feature by default for ensure the security. Neutron support to disable it completely, here it is : Port-Security in port or network.


## Basic Usage

> Here is a sample which help you setup/use port-security.

#### How to enable port-security feature

> Here is how to enable port-security in ml2 extension drivers.


```
# Add the below line in /etc/neutron/plugins/ml2/ml2_conf.ini
# and restart neutron-server.
extension_drivers = port_security
```

#### How to use port-security

> port-security can be disable in Network or Port. 


Create network with port-security false:

```
# neutron net-create  --port_security_enabled=false private_test
# neutron net-show e6c81b80-3dc9-4515-9123-0d17cefd22dc
+-------------------------+--------------------------------------+
| Field                   | Value                                |
+-------------------------+--------------------------------------+
| admin_state_up          | True                                 |
| availability_zone_hints |                                      |
| availability_zones      | nova                                 |
| created_at              | 2016-09-07T07:28:50                  |
| description             |                                      |
| id                      | e6c81b80-3dc9-4515-9123-0d17cefd22dc |
| ipv4_address_scope      |                                      |
| ipv6_address_scope      |                                      |
| mtu                     | 1450                                 |
| name                    | private_test                         |
| port_security_enabled   | False                                |
| qos_policy_id           |                                      |
| router:external         | False                                |
| shared                  | False                                |
| status                  | ACTIVE                               |
| subnets                 | 0e12f67a-962a-4737-85d3-e6b17b63ccc8 |
| tags                    |                                      |
| tenant_id               | 5e336f24f89c43f5ab4d1dd07f54b86d     |
| updated_at              | 2016-09-07T07:28:50                  |
+-------------------------+--------------------------------------+
```

You must boot vm without security-group, then IN and OUT of ipstables rules of the VM will be similiar like following, It will be ACCEPT for all traffic, for sure, you will not go into spoofing iptables chain here.
If the network is port-security disabled, there will not be any rules in iptables.
If the port is updated to port-security disabled, it will has iptables rules as following.

```
Chain neutron-openvswi-FORWARD (1 references)
 pkts bytes target     prot opt in     out     source               destination         
 2900  323K neutron-openvswi-scope  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
   13  2006 ACCEPT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            PHYSDEV match --physdev-out tapa45e21f7-86 --physdev-is-bridged /* Accept all packets when port security is disabled. */
   11  1212 ACCEPT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            PHYSDEV match --physdev-in tapa45e21f7-86 --physdev-is-bridged /* Accept all packets when port security is disabled. */
```

You can also disable port-security in port level, when you disable it, you also need remove security-groups on that port:

```
# neutron port-update --port_security_enabled=false --no-security-groups a45e21f7-8654-495f-b3b9-ea42d73778b4
# neutron port-show a45e21f7-8654-495f-b3b9-ea42d73778b4
+-----------------------+----------------------------------------------------------------------------------------+
| Field                 | Value                                                                                  |
+-----------------------+----------------------------------------------------------------------------------------+
| admin_state_up        | True                                                                                   |
| allowed_address_pairs |                                                                                        |
| binding:vnic_type     | normal                                                                                 |
| created_at            | 2016-09-07T05:49:39                                                                    |
| description           |                                                                                        |
| device_id             | 7a03c4fd-653d-4fc6-a32e-e62f40a38bcd                                                   |
| device_owner          | compute:nova                                                                           |
| dns_name              |                                                                                        |
| extra_dhcp_opts       |                                                                                        |
| fixed_ips             | {"subnet_id": "b8da86ab-6736-4c2c-8989-83d2c7a502f6", "ip_address": "200.200.200.109"} |
| id                    | a45e21f7-8654-495f-b3b9-ea42d73778b4                                                   |
| mac_address           | fa:16:3e:a9:ee:4b                                                                      |
| name                  |                                                                                        |
| network_id            | f0c38656-ac54-483c-b19b-1d2ffde481ed                                                   |
| port_security_enabled | False                                                                                  |
| qos_policy_id         |                                                                                        |
| security_groups       |                                                                                        |
| status                | ACTIVE                                                                                 |
| tenant_id             | 5e336f24f89c43f5ab4d1dd07f54b86d                                                       |
| updated_at            | 2016-09-07T07:02:15                                                                    |
+-----------------------+----------------------------------------------------------------------------------------+
```

## Refer
[Neutron Kilo Spec for port-security](https://specs.openstack.org/openstack/neutron-specs/specs/kilo/ml2-ovs-portsecurity.html)
