**ISP**
```
hostnamectl set-hostname isp
mkdir -p /etc/net/ifaces/ens20
mkdir -p /etc/net/ifaces/ens21
mkdir -p /etc/net/ifaces/ens22
echo -e "TYPE=eth\nBOOTPROTO=dhcp" > /etc/net/ifaces/ens20/options
echo -e "DISABLED=no\nTYPE=eth\nBOOTPROTO=static\nCONFIG_IPv4=yes" > /etc/net/ifaces/ens21/options
echo -e "DISABLED=no\nTYPE=eth\nBOOTPROTO=static\nCONFIG_IPv4=yes" > /etc/net/ifaces/ens22/options
echo "172.16.1.1/28" > /etc/net/ifaces/ens21/ipv4address
echo "172.16.2.1/28" > /etc/net/ifaces/ens22/ipv4address
echo "net.ipv4.ip_forward = 1" >> /etc/net/sysctl.conf
systemctl restart network
apt-get update && apt-get install iptables -y
apt-get reinstall tzdata
timedatectl set-timezone Asia/Yekaterinburg
iptables -t nat -A POSTROUTING -o ens20 -s 0/0 -j MASQUERADE
iptables-save > /etc/sysconfig/iptables
systemctl enable --now iptables
exec bash
```


**HQ-RTR**
```
en
conf t
hostname hq-rtr
ip domain-name au-team.irpo 
interface int0
ip nat outside
ip address 172.16.1.4/28 
exit 
port te0 
service-instance te0/int0
encapsulation untagged 
interface int0
connect port te0 service-instance te0/int0
exit
interface int1
ip nat inside
ip address 192.168.1.1/27
exit
interface int2
ip nat inside
ip address 192.168.2.1/28
exit
interface int3
ip nat inside
ip address 192.168.99.1/29
ex
port te1
service-instance te1/int1
encapsulation dot1q 100 
rewrite pop 1 
exit 
service-instance te1/int2
encapsulation dot1q 200
rewrite pop 1
exit
service-instance te1/int3
encapsulation dot1q 999
rewrite pop 1
ex
ex
interface int1
ᅠ ᅠᅠ ᅠᅠ ᅠconnect port te1 service-instance te1/int1
ᅠ ᅠᅠ ᅠᅠ ᅠexit
ᅠ ᅠᅠ ᅠinterface int2
ᅠ ᅠᅠ ᅠᅠ ᅠconnect port te1 service-instance te1/int2
ᅠ ᅠᅠ ᅠᅠ ᅠexit
ᅠ ᅠᅠ ᅠinterface int3
ᅠ ᅠᅠ ᅠᅠ ᅠconnect port te1 service-instance te1/int3
ᅠ ᅠᅠ ᅠᅠ ᅠexit
ᅠ ᅠᅠ ᅠip route 0.0.0.0 0.0.0.0 172.16.1.1
ᅠ ᅠᅠ ᅠip name-server 8.8.8.8
ᅠ ᅠᅠ ᅠip nat pool NAT_POOL 192.168.1.1-192.168.1.254,192.168.2.1-192.168.2.254
ᅠ ᅠᅠ ᅠip nat source dynamic inside-to-outside pool NAT_POOL overload interface int0
ᅠ ᅠᅠ ᅠint tunnel.0 
ᅠ ᅠᅠ ᅠᅠ ᅠip address 172.16.0.1/30
ᅠ ᅠᅠ ᅠᅠ ᅠip mtu 1400 
ᅠ ᅠᅠ ᅠᅠ ᅠip tunnel 172.16.1.4 172.16.2.5 mode gre 
ᅠ ᅠᅠ ᅠᅠ ᅠip ospf authentication-key ecorouter
ᅠ ᅠᅠ ᅠᅠ ᅠex
ᅠ ᅠᅠ
ᅠ ᅠᅠ ᅠrouter ospf 1 
ᅠ ᅠᅠ ᅠᅠ ᅠ  network 172.16.0.0/30 area 0 
  ᅠ ᅠᅠ ᅠᅠ ᅠnetwork 192.168.1.0/27 area 0
  ᅠ ᅠᅠ ᅠᅠ ᅠnetwork 192.168.2.0/28 area 0
  ᅠ ᅠᅠ ᅠᅠ ᅠpassive-interface default
  ᅠ ᅠᅠ ᅠᅠ ᅠno passive-interface tunnel.0 
 ᅠ ᅠᅠ ᅠᅠ ᅠ area 0 authentication
  ᅠ ᅠᅠ ᅠᅠ ᅠex
 ᅠ ᅠᅠ ᅠip pool cli_pool 192.168.2.10-192.168.2.10
ᅠ ᅠᅠ ᅠdhcp-server 1
ᅠ ᅠᅠ ᅠᅠ ᅠpool cli_pool 1 
ᅠ ᅠᅠ ᅠᅠ ᅠmask 255.255.255.240 
ᅠ ᅠᅠ ᅠᅠ ᅠgateway 192.168.2.1 
ᅠ ᅠᅠ ᅠᅠ ᅠdns 192.168.1.10 
ᅠ ᅠᅠ ᅠᅠ ᅠdomain-name au-team.irpo
ᅠ ᅠᅠ ᅠᅠ ᅠex

ᅠ ᅠᅠ ᅠinterface int2
ᅠ ᅠᅠ ᅠᅠ ᅠdhcp-server 1
ᅠ ᅠᅠ ᅠᅠ ᅠex
ᅠ ᅠᅠ ᅠntp timezone utc+5
        username net_admin
ᅠ ᅠᅠ ᅠᅠ ᅠpassword P@ssw0rd 
ᅠ ᅠᅠ ᅠᅠ ᅠrole admin 
ᅠ ᅠᅠ ᅠᅠ ᅠex
ᅠ ᅠᅠ ᅠend
ᅠ ᅠwrite memory
```

**BR-RTR**
```
en
ᅠ ᅠconf t
ᅠ ᅠᅠ ᅠhostname br-rtr 
ᅠ ᅠᅠ ᅠip domain-name au-team.irpo
ᅠ ᅠᅠ ᅠinterface int0
ᅠ ᅠᅠ ᅠᅠ ᅠip nat outside
ᅠ ᅠᅠ ᅠᅠ ᅠip address 172.16.2.5/28
ᅠ ᅠᅠ ᅠᅠ ᅠexit
ᅠ ᅠᅠ ᅠport te0
ᅠ ᅠᅠ ᅠᅠ ᅠservice-instance te0/int0
ᅠ ᅠᅠ ᅠᅠ ᅠᅠ ᅠencapsulation untagged
ᅠ ᅠᅠ ᅠᅠ ᅠᅠ ᅠexit
ᅠ ᅠᅠ ᅠᅠ ᅠexit
ᅠ ᅠᅠ ᅠinterface int0
ᅠ ᅠᅠ ᅠᅠ ᅠconnect port te0 service-instance te0/int0
ᅠ ᅠᅠ ᅠᅠ ᅠexit
ᅠᅠ ᅠ ᅠinterface int1
ᅠ ᅠᅠ ᅠᅠ ᅠip nat inside
ᅠ ᅠᅠᅠ ᅠ ᅠip address 192.168.3.1/28
ᅠ ᅠᅠᅠ ᅠ ᅠexit
ᅠᅠ ᅠ ᅠport te1
ᅠ ᅠᅠ ᅠ ᅠ ᅠservice-instance te1/int1
ᅠ ᅠᅠ ᅠ ᅠᅠᅠ ᅠencapsulation untagged
ᅠ ᅠᅠ ᅠᅠ ᅠᅠ ᅠexit
ᅠ ᅠᅠᅠ ᅠ ᅠ  exit
ᅠᅠ ᅠ ᅠinterface int1
ᅠ ᅠᅠ ᅠᅠ ᅠ  connect port te1 service-instance te1/int1
ᅠ ᅠᅠᅠ ᅠ ᅠ  exit
ᅠ ᅠᅠ ᅠip route 0.0.0.0 0.0.0.0 172.16.2.1
		ip name-server 8.8.8.8
ᅠ ᅠᅠ ᅠip nat pool NAT_POOL 192.168.3.1-192.168.3.254
ᅠ ᅠᅠ ᅠip nat source dynamic inside-to-outside pool NAT_POOL overload interface int0
ᅠ ᅠᅠ ᅠint tunnel.0 
ᅠ ᅠᅠ ᅠᅠ ᅠip address 172.16.0.2/30
ᅠ ᅠᅠ ᅠᅠ ᅠip mtu 1400 
ᅠ ᅠᅠ ᅠᅠ ᅠip tunnel 172.16.2.5 172.16.1.4 mode gre
ᅠ ᅠᅠ ᅠᅠ ᅠip ospf authentication-key ecorouter 
ᅠ ᅠᅠ ᅠᅠ ᅠex
ᅠ ᅠᅠ ᅠ  router ospf 1 
ᅠ ᅠᅠ ᅠᅠ ᅠ  network 172.16.0.0/30 area 0 
  ᅠ ᅠᅠ ᅠᅠ ᅠnetwork 192.168.3.0/28 area 0
  ᅠ ᅠᅠ ᅠᅠ ᅠpassive-interface default 
  ᅠ ᅠᅠ ᅠᅠ ᅠno passive-interface tunnel.0 
 ᅠ ᅠᅠ ᅠᅠ ᅠ area 0 authentication 
 ᅠ ᅠᅠ ᅠᅠ ᅠex
ᅠ ᅠᅠ ᅠ
ᅠ ᅠᅠ ᅠntp timezone utc+5
ᅠ ᅠᅠ ᅠusername net_admin 
ᅠ ᅠᅠ ᅠᅠ ᅠpassword P@ssw0rd 
ᅠ ᅠᅠ ᅠᅠ ᅠrole admin
ᅠ ᅠᅠ ᅠᅠ ᅠend
ᅠ ᅠwrite memory
```
**BR-SRV**

```
hostnamectl set-hostname br-srv.au-team.irpo;
mkdir -p /etc/net/ifaces/ens20
echo -e "\nDISABLED=no\nTYPE=eth\nBOOTPROTO=static\nCONFIG_IPv4=yes" > /etc/net/ifaces/ens20/options
echo "192.168.3.10/28" > /etc/net/ifaces/ens20/ipv4address
echo "default via 192.168.3.1" > /etc/net/ifaces/ens20/ipv4route
echo "nameserver 8.8.8.8" > /etc/resolv.conf 
systemctl restart network
useradd sshuser -u 2026
useradd -p P@ssw0rd sshuser
gpasswd -a "sshuser" wheel
echo -e "WHEEL_USERS ALL=(ALL:ALL) NOPASSWD: ALL" > /etc/sudoers
echo "Port 2026" >> /etc/openssh/sshd_config
echo "MaxAuthTries 2" >> /etc/openssh/sshd_config
echo "PasswordAuthentication yes" >> /etc/openssh/sshd_config
echo "Banner /etc/openssh/banner" >> /etc/openssh/sshd_config
echo "Authorized access only" >> /etc/openssh/banner
systemctl restart sshd
timedatectl set-timezone Asia/Yekaterinburg
exec bash
```

**HQ-CLI**
```
hostnamectl set-hostname hq-cli.au-team.irpo
mkdir -p /etc/net/ifaces/ens20
echo -e "\nTYPE=eth\nBOOTPROTO=dhcp" > /etc/net/ifaces/ens20/options
systemctl restart network
timedatectl set-timezone Asia/Yekaterinburg
exec bash
```

**HQ-SRV**
```
hostnamectl set-hostname hq-srv.au-team.irpo;
mkdir -p /etc/net/ifaces/ens20
echo -e "\nDISABLED=no\nTYPE=eth\nBOOTPROTO=static\nCONFIG_IPv4=yes" > /etc/net/ifaces/ens20/options
echo "192.168.1.10/27" > /etc/net/ifaces/ens20/ipv4address
echo "default via 192.168.1.1" > /etc/net/ifaces/ens20/ipv4route
echo "nameserver 8.8.8.8" > /etc/resolv.conf 
systemctl restart network
useradd sshuser -u 2026
useradd -p P@ssw0rd sshuser
gpasswd -a "sshuser" wheel
echo -e "WHEEL_USERS ALL=(ALL:ALL) NOPASSWD: ALL" > /etc/sudoers
echo "Port 2026" >> /etc/openssh/sshd_config
echo "MaxAuthTries 2" >> /etc/openssh/sshd_config
echo "PasswordAuthentication yes" >> /etc/openssh/sshd_config
echo "Banner /etc/openssh/banner" >> /etc/openssh/sshd_config
echo "Authorized access only" >> /etc/openssh/banner
systemctl restart sshd
timedatectl set-timezone Asia/Yekaterinburg
systemctl restart network
apt-get update && apt-get install dnsmasq
systemctl enable --now dnsmasq
echo "no-resolv" >> /etc/dnsmasq.conf
echo "domain=au-team.irpo" >> /etc/dnsmasq.conf
echo "server=8.8.8.8" >> /etc/dnsmasq.conf
echo "interface=*" >> /etc/dnsmasq.conf
echo "address=/docker.au-team.irpo/172.16.1.1" >> /etc/dnsmasq.conf
echo "address=/web.au-team.irpo/172.16.2.1" >> /etc/dnsmasq.conf
echo "address=/address=/hq-rtr.au-team.irpo/192.168.1.1" >> /etc/dnsmasq.conf
echo "address=/ptr-record=1.1.168.192.in-addr.arpa,hq-rtr.au-team.irpo" >> /etc/dnsmasq.conf
echo "address=/hq-srv.au-team.irpo/192.168.1.10" >> /etc/dnsmasq.conf
echo "ptr-record=10.1.168.192.in-addr.arpa,hq-srv.au-team.irpo" >> /etc/dnsmasq.conf
echo "address=/hq-cli.au-team.irpo/192.168.2.10" >> /etc/dnsmasq.conf
echo "ptr-record=10.2.168.192.in-addr.arpa,hq-cli.au-team.irpo" >> /etc/dnsmasq.conf
echo "address=/br-rtr.au-team.irpo/192.168.3.1" >> /etc/dnsmasq.conf
echo "address=/br-srv.au-team.irpo/192.168.3.10" >> /etc/dnsmasq.conf
echo "192.168.1.1   hq-rtr.au-team.irpo" >> /etc/hosts
systemctl enable --now dnsmasq
systemctl restart dnsmasq
exec bash
```











echo "User_Alias	WHEEL_USERS = %wheel" >> /etc/sudoers
echo "User_Alias	XGRP_USERS = %xgrp" >> /etc/sudoers
echo "Defaults:XGRP_USERS env_keep += "DISPLAY XAITHORITY" " >> /etc/sudoers
echo "WHEEL_USERS ALL=(ALL:ALL) NOPASSWD: ALL" >> /etc/sudoers
