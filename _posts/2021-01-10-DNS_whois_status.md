---
title: DevOps -- DNS Domain Status
---

> 今天发现tp link的ddns不生效，发现是其域名处于了clientHold状态。

DNS的状态信息可以通过whois命令查询：whois tpddns.cn


```
whois:        whois.cnnic.cn

status:       ACTIVE
remarks:      Registration information: http://www.cnnic.cn/

created:      1990-11-28
changed:      2018-03-01
source:       IANA

Domain Name: tpddns.cn
ROID: 20150327s10001s75385635-cn
Domain Status: clientTransferProhibited
Domain Status: clientHold
Registrant: 普联技术有限公司
Registrant Contact Email: liuxueting@tp-link.com.cn
Sponsoring Registrar: 北京新网数码信息技术有限公司
Name Server: ns1.tpddns.cn
Name Server: ns2.tpddns.cn
Registration Time: 2015-03-27 15:47:48
Expiration Time: 2023-03-27 15:47:48
DNSSEC: unsigned
```


可以看到Domain Status是：
```
Domain Status: clientTransferProhibited
Domain Status: clientHold
```

如果status处于“clientHold”或“serverHold”， 一般是实名认证问题。
