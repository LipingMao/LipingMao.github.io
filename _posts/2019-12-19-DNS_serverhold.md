---
title: DNS -- 域名实名认证
---

> 在国内的域名是需要进行实名认证的，如果没有进行实名认证，“注册局”会将Domain Status变为serverHold。这种情况会出现在新加域名，但还未实名认证。也可能出现在域名从国外转入国内的时候（网站备案要求域名在国内实名认证过）。


以下过程为发生在我们将域名从国外转入国内时，我们大约在12/22提交了转域名的申请，大约过了1周时间，域名转入国内，可以从whois pano.video中查看到域名信息，此处可以看到“2019-12-29 13:11:49”发生了update，但是此时域名的状态进入了“serverHold”状态，如下所示：
```
Domain Name:pano.video
Registrar WHOIS Server:whois.dnspod.com
Registrar URL:https://www.dnspod.com/
Updated Date:2019-12-29 13:11:49
Creation Date:2019-09-16 14:20:31
Registrar Registration Expiration Date:2021-09-16 14:20:31
Registrar:DNSPod, Inc.
Registrar IANA ID:1697
Registrar Abuse Contact Email:cloud_abuse@tencent.com
Registrar Abuse Contact Phone:+86.4009100100-9
Domain Status:serverHold https://www.icann.org/epp#serverHold
Name Server:ns-1382.awsdns-44.org
Name Server:ns-1546.awsdns-01.co.uk
Name Server:ns-620.awsdns-13.net
Name Server:ns-99.awsdns-12.com
DNSSEC: unsigned
URL of the ICANN WHOIS Data Problem Reporting System: http://wdprs.internic.net
```

“serverHold”状态是“注册局”定义的一个状态，一般来说这是因为 ：注册局设置暂停解析（大多原因是.com、.cn、.net 等域名未进行实名认证，完成实名审核后自动解除该状态）。  [1]

虽然转入前已经进行过实名认证，但是腾讯云的解释是：如果域名是从国外转入国内，注册局会有一段时间来同步实名认证的状态，因此在这段时间内域名会出于serverHold状态，大概12-24小时后，域名状态恢复正常，如下所示：
```
Domain Name:pano.video
Registrar WHOIS Server:whois.dnspod.com
Registrar URL:https://www.dnspod.com/
Updated Date:2019-12-30 09:01:41
Creation Date:2019-09-16 14:20:31
Registrar Registration Expiration Date:2021-09-16 14:20:31
Registrar:DNSPod, Inc.
Registrar IANA ID:1697
Registrar Abuse Contact Email:cloud_abuse@tencent.com
Registrar Abuse Contact Phone:+86.4009100100-9
Domain Status:ok https://www.icann.org/epp#ok
Name Server:ns-1382.awsdns-44.org
Name Server:ns-1546.awsdns-01.co.uk
Name Server:ns-620.awsdns-13.net
Name Server:ns-99.awsdns-12.com
DNSSEC: unsigned
URL of the ICANN WHOIS Data Problem Reporting System: http://wdprs.internic.net
```


### Refer

[1] https://cloud.tencent.com/document/product/242/7924
