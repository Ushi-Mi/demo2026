**BR-SRV**
```
apt-get update && apt-get install task-samba-dc -y
apt-get install sudo-samba-schema -y
apt-get install chrony -y
apt-get install docker-compose docker-engine -y
apt-get install ansible -y 
rm -rf /etc/samba/smb.conf 
echo "nameserver 192.168.1.10" >> /etc/resolv.conf

echo "192.168.3.10     br-srv.au-team.irpo" >> vim /etc/hosts 
samba-tool domain provision



samba-tool domain provision --AU-TEAM.IRPO --AU-TEAM --dc --SAMBA_INTERNAL --192.168.1.10 --P@ssw0rd --P@ssw0rd




mv -f /var/lib/samba/private/krb5.conf /etc/krb5.conf 
systemctl enable --now samba 
samba-tool user add hquser1 P@ssw0rd 
samba-tool user add hquser2 P@ssw0rd 
samba-tool user add hquser3 P@ssw0rd 
samba-tool user add hquser4 P@ssw0rd 
samba-tool user add hquser5 P@ssw0rd 
samba-tool group add hq 
samba-tool group addmembers hq hquser1,hquser2,hquser3,hquser4,hquser5

apt-repo add rpm http://altrepo.ru/local-p10 noarch local-p10 
apt-get update && apt-get install sudo-samba-schema -y 



```

**HQ-SRV**
```
echo "erver=/au-team.irpo/192.168.3.10" >> /etc/dnsmasq.conf 
systemctl restart dnsmasq 
apt-get update && apt-get install fdisk -y 
```

