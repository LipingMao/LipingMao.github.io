---
title: LibreVPN -- How to
---

## What is site to site VPN?

Site to site VPN extends the company's network from one location to multiple locations.  This wiki is used for how to build site to site VPN on CentOS7


## How to build site to site VPN in cloud?

run the following scripts to install libreswan / iptables services and update sysctl, and iptables rules. In this sample, we are using 10.0.0.0/8 as our global internal network CIDR.
```
yum install -y openswan lsof iptables-services
 
function systctl_set() {
  key=$1
  value=$2
  config_path='/etc/sysctl.conf'
  # set now
  sysctl -w ${key}=${value}
  # persist on reboot
  if grep -q "${key}" "${config_path}"; then
    sed -i "s/^${key}.*$/${key} = ${value}/" "${config_path}"
  else
    echo "${key} = ${value}" >> "${config_path}"
  fi
}
 
if [ "$(getconf LONG_BIT)" = "64" ]; then
  SHM_MAX=68719476736
  SHM_ALL=4294967296
else
  SHM_MAX=4294967295
  SHM_ALL=268435456
fi
systctl_set kernel.msgmnb 65536
systctl_set kernel.msgmax 65536
systctl_set kernel.shmmax $SHM_MAX
systctl_set kernel.shmall $SHM_ALL
systctl_set net.ipv4.ip_forward 1
systctl_set net.ipv4.conf.all.accept_source_route 0
systctl_set net.ipv4.conf.all.accept_redirects 0
systctl_set net.ipv4.conf.all.send_redirects 0
systctl_set net.ipv4.conf.all.rp_filter 0
systctl_set net.ipv4.conf.default.accept_source_route 0
systctl_set net.ipv4.conf.default.accept_redirects 0
systctl_set net.ipv4.conf.default.send_redirects 0
systctl_set net.ipv4.conf.default.rp_filter 0
sysctl -p
 
 
# using 10.0.0.0/8 for SNAT
iptables -P FORWARD ACCEPT
iptables -t nat -A POSTROUTING -s 10.0.0.0/8 -j MASQUERADE
 
systemctl enable iptables
iptables-save > /etc/sysconfig/iptables
 
systemctl enable ipsec
```

Configure ipsec in both side, here is an example, Hub Configuration:
```
/etc/ipsec.d/awsbj-awsnx.conf
conn awsbj-awsnx
    type=tunnel
    authby=secret
    auto=start
    replay-window=0
    left=%defaultroute
    leftid=54.222.190.251                           <--- LOCAL EIP
        leftsubnet=10.0.0.0/8                       <--- Global IP RANGE
    leftnexthop=%defaultroute                  
        right=52.83.79.32                           <--- REMOTE EIP
        rightsubnet=10.0.0.0/20                     <--- REMOTE IP RANGE
    pfs=yes
    ike=aes128-sha1;modp1024
    ikelifetime=28800s
    esp=aes128-sha1;modp1024
    salifetime=28800s
 
conn exclude-local
        left=10.0.128.173                           <--- LOCAL IP
        leftsubnet=10.0.128.0/20                    <--- LOCAL VPC IP RANGE
        right=0.0.0.0
        rightsubnet=10.0.128.0/20                   <--- LOCAL VPC IP RANGE
        authby=never
        type=passthrough
        auto=route
 
/etc/ipsec.d/awsbj-awsnx.secrets
54.222.190.251 52.83.79.32: PSK "$YOURPASSWORD"     <--- LOCAL EIP , REMOTE EIP, Pre shared Key
```

Spoke Configuration:
```
conn awsnx-awsbj
    type=tunnel
    authby=secret
    auto=start
    left=%defaultroute
    leftid=52.83.79.32                       <--- LOCAL EIP
        leftsubnet=10.0.0.0/20               <--- LOCAL VPC IP RANGE
    leftnexthop=%defaultroute
        right=54.222.190.251                 <--- REMOTE EIP
        rightsubnet=10.0.0.0/8               <--- GLOBAL IP RANGE
        pfs=yes
        ike=aes128-sha1;modp1024
        ikelifetime=28800s
        esp=aes128-sha1;modp1024
        salifetime=28800s
 
conn exclude-local
        left=10.0.0.83                      <--- LOCAL IP
        leftsubnet=10.0.0.0/20              <--- LOCAL VPC IP RANGE
        right=0.0.0.0
        rightsubnet=10.0.0.0/20             <--- LOCAL VPC IP RANGE
        authby=never
        type=passthrough
        auto=route
 
 
/etc/ipsec.d/awsnx-awsbj.secrets
52.83.79.32 54.222.190.251: PSK "$YOURPASSWORD"     <--- LOCAL EIP , REMOTE EIP, Pre shared Key
```

After configurated both side, run "ipsec start" in both side to start ipsec, and run ipsec status to check status. Once it is done, you should able to ping vms in other side as local network.
