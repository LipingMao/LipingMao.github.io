---
title: OpenVPN -- Install, LDAP, Push DNS
---
# OpenVPN

When we use Cloud environment, we usually need an Internal Network Environment. We can use OpenVPN as client - Site VPN. This blog will introduce how to install OpenVPN, integration with LDAP, and push DNS stuff.



## Install

You can refer to https://www.golinuxcloud.com/install-openvpn-server-easy-rsa-3-centos-7/ , It has a clear step by step guide for install OpenVPN on CentOS.



## Integration with LDAP

A typical requirement is using LDAP user/pass to login OpenVPN, but not case by case generate client Key. Here is steps to enable LDAP in OpenVPN:

Install openvpn auth ldap:

```
yum -y install openvpn-auth-ldap
```



Add the following LDAP configuration in /etc/openvpn/server.conf:

```
plugin /usr/lib64/openvpn/plugin/lib/openvpn-auth-ldap.so /etc/openvpn/auth/ldap.conf
```



Update LDAP Configuration in /openvpn/auth/ldap.conf:

```
<LDAP>
      URL             ldap://10.0.4.89:389
      BindDN          "cn=openvpn.gen,ou=Generics,ou=Pano Users,dc=panorama,dc=com"
      Password        "$PASSWORD"
      Timeout         15
      TLSEnable       no
      FollowReferrals no
</LDAP>

<Authorization>
      BaseDN          "ou=Pano Users,dc=panorama,dc=com"
      SearchFilter    "(&(cn=%u)(memberof=cn=vpn-bj,ou=VPN Groups,ou=Pano Groups,dc=panorama,dc=com))"
      RequireGroup    false
</Authorization>
```



Note:

There is bug when enable RequireGroup now.



In Client ovpn file, add the following lines: 

```
auth-user-pass
remote-cert-tls server
```



##Push DNS Server

Why we want to push DNS server in OpenVPN?Because we want to have our internal DNS server to resolve Internal DNS (10.X.Y.Z in internal), so that you can use hostname to login the server.

In this sample, we configure our internal DNS server in first (A.B.C.D), Tencent DNS as Second/Third  (183.60.83.19 , 183.60.82.98)

```
push "dhcp-option DNS A.B.C.D"
push "dhcp-option DNS 183.60.83.19"
push "dhcp-option DNS 183.60.82.98"
```