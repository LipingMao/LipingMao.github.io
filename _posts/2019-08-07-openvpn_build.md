---
title: DevOps -- Build OpenVPN in AWS
---

## Background

> When you have VPC in cloud, typically, you need client VPN to connect the VPC, so that you can use internal ip to access the internal services within VPC.

## Install Steps

Here a step by step wiki to help people to build OpenVPN in AWS VPC environment. My AWS VPC is using 10.0.0.0/16 network.

1. Build one instane with CentOS 7 in AWS public subnet and assign EIP for the instance, make sure allowed udp port 1194 which is used for udp traffic.

2. Install RPMs:
```
yum -y install epel-release
yum -y install openvpn easy-rsa iptables-services
yum -y upgrade
```

3. Configure easy-rsa:
```
mkdir -p /etc/openvpn/easy-rsa/
cd /usr/share/easy-rsa/3
cp -rf * /etc/openvpn/easy-rsa/
cd /etc/openvpn/easy-rsa/
./easyrsa init-pki
./easyrsa build-ca
./easyrsa gen-dh
```

4. Generate server rsa:
```
./easyrsa gen-req bastion nopass
./easyrsa sign server  bastion
mkdir /etc/openvpn/keys/
chmod 750 /etc/openvpn/keys
cp -a /etc/openvpn/easy-rsa/pki/ca.crt /etc/openvpn/keys/
cp -a /etc/openvpn/easy-rsa/pki/dh.pem /etc/openvpn/keys/dh2048.pem
cp -a /etc/openvpn/easy-rsa/pki/issued/bastion.crt /etc/openvpn/keys/
cp -a /etc/openvpn/easy-rsa/pki/private/bastion.key /etc/openvpn/keys/
```

5. Generate client key:
```
cd /etc/openvpn/easy-rsa
./easyrsa gen-req limao nopass
./easyrsa sign client limao
cp -a /etc/openvpn/easy-rsa/pki/issued/limao.crt /etc/openvpn/keys/
cp -a /etc/openvpn/easy-rsa/pki/private/limao.key /etc/openvpn/keys/
```

7. Configure openvpn server:
```
port 1194
proto udp
dev tun
comp-lzo
management 127.0.0.1 1194
keepalive 10 120
persist-key
persist-tun
ifconfig-pool-persist ipp.txt
status openvpn-status.log
verb 3
server 172.16.0.0 255.255.255.0
push "route 10.0.0.0 255.255.0.0"
push "dhcp-option DNS 10.0.0.2"
push "dhcp-option DOMAIN cn-northwest-1.compute.internal"
ca /etc/openvpn/keys/ca.crt
cert /etc/openvpn/keys/bastion.crt
key /etc/openvpn/keys/bastion.key  # This file should be kept secret
dh /etc/openvpn/keys/dh2048.pem
```
Configure /etc/openvpn/server.conf, and change the "route 10.0.0.0 255.255.0.0" to your VPC subnet.

7. Start and enable openvpn service:
```
systemctl -f enable openvpn@server.service
systemctl start openvpn@server.service
```

8. Create client ovpn file:
here is a sample to generate a client file named "etbi".
```
cd /etc/openvpn/easy-rsa
./easyrsa gen-req etbi nopass
./easyrsa sign client etbi
cp -a /etc/openvpn/easy-rsa/pki/issued/etbi.crt /etc/openvpn/keys/
cp -a /etc/openvpn/easy-rsa/pki/private/etbi.key /etc/openvpn/keys/
cd /etc/openvpn/keys/
sh merge.sh etbi
```
Here is the /etc/openvpn/keys/merge.sh
```
cat client.conf > ${1}.ovpn
echo "" >> ${1}.ovpn
echo "<cert>" >> ${1}.ovpn
cat ${1}.crt >> ${1}.ovpn
echo "</cert>" >> ${1}.ovpn
echo "" >> ${1}.ovpn
echo "<key>" >> ${1}.ovpn
cat ${1}.key >> ${1}.ovpn
echo "</key>" >> ${1}.ovpn
echo "" >> ${1}.ovpn
echo "<ca>" >> ${1}.ovpn
cat ca.crt >> ${1}.ovpn
echo "</ca>" >> ${1}.ovpn
```
Here is /etc/openvpn/keys/client.conf, you need to replace 52.83.79.32 to your EIP.
```
client
nobind
dev tun
comp-lzo
remote 52.83.79.32 1194 udp  ## NOTE: use your IP replace this
```

9. network forward and iptables
```
vi /etc/sysctl.conf
net.ipv4.ip_forward = 1
sysctl -p
iptables -t nat -A POSTROUTING -s 172.16.0.0/24 -j MASQUERADE
service iptables save
service iptables restart
systemctl enable iptables
systemctl restart iptables
service iptables save
```

10. Configure OpenVPN client on Mac.
Download the client from [openvpn mac client](https://openvpn.net/downloads/openvpn-connect-v2-macos.dmg)
You can configure the OpenVPN client with import from etbi.ovpn file.
