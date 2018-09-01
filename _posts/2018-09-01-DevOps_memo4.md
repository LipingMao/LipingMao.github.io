---
title: shared network to RBAC
---

## Background

> 在产线环境中配置了shared network网络，但是由于是默认所有租户可见，导致网络资源ip使用不可控。有需求将已有的shared网络改为某些租户可见的网络。


## How to do it?

可以利用neutron network的rbac功能，见[refer](https://docs.openstack.org/newton/networking-guide/config-rbac.html)。

### #Step 1

neutron rbac 默认情况下quota是10，也就是说默认情况写下设置的rbac条目不能超过10条。需要首先把quota扩大，如果quota设置为负数，则没有限制。不过目前newton版本在neutron client端有以下[bug](https://review.openstack.org/#/c/383531/3)。Fix之后，可以使用以下命令修改quota:

```
#neutron quota-update --rbac-policy -1
```

### #Step 2

如果neutron网络已经被这个租户使用，neutron是不允许修改成这个网络对此租户不可见的。换句话说，需要配置所有已经使用这个网络的租户的rbac。假设租户的id记录在tenants文件中，network是provider网络。则使用以下命令可以配置rbac(NETWORK_UUID 应替换成网络UUID):

```
#cat tenants | xargs -I % neutron  rbac-create --type network --target-tenant % --action access_as_shared $NETWORK_UUID
```

### #Step 3

值得注意的是，rbac是白名单，所有没有配置的租户对此网络都不可见。新建租户要看到网络需要手动配置network的rbac。
