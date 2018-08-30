---
title: DevOps Memo(2) -- DHCP client request rfc3442-classless-static-routes option
---

## Background

> 在产线上，我们的neutron同一个network添加多个subnet后，vm没有获取多subnet的路由。


## What is the problem?

VM在OpenStack Neutron环境是通过dhcp获取IP地址，默认路由，dns server等信息。DHCP Server通过option返回给VM这些信息。产线上同一个Neutron network 可以添加多个subnet网段。<br>
假设添加的网段是: <br>
192.168.0.1/24<br>
192.168.1.1/24<br>
VM的地址为192.168.0.100，当添加了192.168.1.1/24网段后，VM应该获取到以下路由:<br>
192.168.0.0/24 via eth0<br>
192.168.1.0/24 via eth0<br>
但是VM在dhcp renew之后也没有获取到新加的网络。


## What is the root cause?

查看neutron dhcp agent端代码，dnsmasq已经将rfc3442-classless-static-routes正确配置。在vm中查看eth0的dhcp lease文件发现，option已经更新，但是没有update到vm。基本定位是vm内部dhcpclient的问题。
查看network-script代码，在dhcp renew时处理的逻辑有问题，发现以下[bug](https://bugzilla.redhat.com/show_bug.cgi?id=1363791)。


## What is the solution?

升级至dhcp-4.2.5-59.el7以上版本可以解决问题。
