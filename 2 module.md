**2.1-2.5 включительно +2.8**

**BR-SRV**
```
if ! grep -q '^nameserver 8\.8\.8\.8$' /etc/resolv.conf; then
    echo 'nameserver 8.8.8.8' | sudo tee -a /etc/resolv.conf
fi
apt-get update && apt-get install wget dos2unix task-samba-dc -y
sleep 3
echo nameserver 192.168.1.10 >> /etc/resolv.conf
sleep 2
echo 192.168.3.10 br-srv.au-team.irpo >> /etc/hosts
rm -rf /etc/samba/smb.conf
samba-tool domain provision --realm=AU-TEAM.IRPO --domain=AU-TEAM --adminpass=P@ssw0rd --dns-backend=SAMBA_INTERNAL --server-role=dc --option='dns forwarder=192.168.1.10'
mv -f /var/lib/samba/private/krb5.conf /etc/krb5.conf
systemctl enable --now samba.service
samba-tool user add hquser1 P@ssw0rd
samba-tool user add hquser2 P@ssw0rd
samba-tool user add hquser3 P@ssw0rd
samba-tool user add hquser4 P@ssw0rd
samba-tool user add hquser5 P@ssw0rd
samba-tool group add hq
samba-tool group addmembers hq hquser1,hquser2,hquser3,hquser4,hquser5
wget https://raw.githubusercontent.com/sudo-project/sudo/main/docs/schema.ActiveDirectory
dos2unix schema.ActiveDirectory
sed -i 's/DC=X/DC=au-team,DC=irpo/g' schema.ActiveDirectory
head -$(grep -B1 -n '^dn:$' schema.ActiveDirectory | head -1 | grep -oP '\d+') schema.ActiveDirectory > first.ldif
tail +$(grep -B1 -n '^dn:$' schema.ActiveDirectory | head -1 | grep -oP '\d+') schema.ActiveDirectory | sed '/^-/d' > second.ldif
ldbadd -H /var/lib/samba/private/sam.ldb first.ldif --option="dsdb:schema update allowed"=true
ldbmodify -v -H /var/lib/samba/private/sam.ldb second.ldif --option="dsdb:schema update allowed"=true
samba-tool ou add 'ou=sudoers'
cat << EOF > sudoRole-object.ldif
dn: CN=prava_hq,OU=sudoers,DC=au-team,DC=irpo
changetype: add
objectClass: top
objectClass: sudoRole
cn: prava_hq
name: prava_hq
sudoUser: %hq
sudoHost: ALL
sudoCommand: /bin/grep
sudoCommand: /bin/cat
sudoCommand: /usr/bin/id
sudoOption: !authenticate
EOF
ldbadd -H /var/lib/samba/private/sam.ldb sudoRole-object.ldif
echo -e "dn: CN=prava_hq,OU=sudoers,DC=au-team,DC=irpo\nchangetype: modify\nreplace: nTSecurityDescriptor" > ntGen.ldif
ldbsearch  -H /var/lib/samba/private/sam.ldb -s base -b 'CN=prava_hq,OU=sudoers,DC=au-team,DC=irpo' 'nTSecurityDescriptor' | sed -n '/^#/d;s/O:DAG:DAD:AI/O:DAG:DAD:AI\(A\;\;RPLCRC\;\;\;AU\)\(A\;\;RPWPCRCCDCLCLORCWOWDSDDTSW\;\;\;SY\)/;3,$p' | sed ':a;N;$!ba;s/\n\s//g' | sed -e 's/.\{78\}/&\n /g' >> ntGen.ldif
ldbmodify -v -H /var/lib/samba/private/sam.ldb ntGen.ldif
apt-get install chrony -y 
echo -e "server 172.16.2.1 iburst prefer" > /etc/chrony.conf
systemctl enable --now chronyd
systemctl restart chronyd
chronyc sources 
timedatectl
apt-get update && apt-get install ansible sshpass -y
echo -e "[s]\nHQ-SRV ansible_host=192.168.1.10\nHQ-CLI ansible_host=192.168.2.10\n[s:vars]\nansible_user=sshuser\nansible_port=2026\nansible_password=P@ssw0rd\n[r]\nHQ-RTR ansible_host=192.168.1.1\nBR-RTR ansible_host=192.168.3.1\n[r:vars]\nansible_user=net_admin\nansible_password=P@ssw0rd\nansible_connection=network_cli\nansible_network_os=ios" > /etc/ansible/hosts
rm -f /etc/ansible/ansible.cfg
echo -e "[defaults]\ninterpreter_python=auto_silent\nhost_key_checking=false" > /etc/ansible/ansible.cfg
ansible all -m ping

```


**HQ-SRV**

**Создаем 2 жестких диска на локальном сайте с объемом по 1 ГБ**

```
echo "server=/au-team.irpo/192.168.3.10" >> /etc/dnsmasq.conf
systemctl restart dnsmasq
apt-get update && apt-get install fdisk -y
mdadm --create /dev/md0 --level=0 --raid-devices=2 /dev/sd[b-d]
fdisk /dev/md0 << EOF
n
p
1
2048
4186111
w
EOF

echo "/dev/md0p1      /raid   ext4    defaults        0       0" >> /etc/hosts
mkdir /raid
mount -a
apt-get install nfs-server -y
mkdir /raid/nfs
chown 99:99 /raid/nfs
chmod 777 /raid/nfs
echo "/raid/nfs 192.168.2.0/28(rw,sync,no_subtree_check)" >> /etc/exports
exportfs -a
exportfs -v
systemctl enable nfs
systemctl restart nfs
apt-get install chrony -y 
echo -e "server 172.16.1.1 iburst prefer" > /etc/chrony.conf
systemctl enable --now chronyd
systemctl restart chronyd
chronyc sources 
timedatectl 
```
**HQ-CLI**
```
sed -i 's/BOOTPROTO=static/BOOTPROTO=dhcp/' /etc/net/ifaces/ens20/options
systemctl restart network
apt-get update && apt-get install bind-utils -y
system-auth write ad AU-TEAM.IRPO cli AU-TEAM 'administrator' 'P@ssw0rd'
reboot
```
```
apt-get install sudo libsss_sudo -y
control sudo public
sed -i '19 a\
sudo_provider = ad' /etc/sssd/sssd.conf
sed -i 's/services = nss, pam/services = nss, pam, sudo/' /etc/sssd/sssd.conf
sed -i '28 a\
sudoers: files sss' /etc/nsswitch.conf
rm -rf /var/lib/sss/db/*
sss_cache -E
systemctl restart sssd

apt-get install nfs-clients -y
mkdir -p /mnt/nfs
echo "192.168.1.10:/raid/nfs  /mnt/nfs        nfs     intr,soft,_netdev,x-systemd.automount   0     0" >> /etc/fstab
mount -a
mount -v
touch /mnt/nfs/test
apt-get install chrony -y 
echo -e "server 172.16.1.1 iburst prefer" > /etc/chrony.conf
systemctl enable --now chronyd
systemctl restart chronyd
chronyc sources 
timedatectl
systemctl restart network
useradd sshuser -u 2026
useradd -p P@ssw0rd sshuser
gpasswd -a "sshuser" wheel
echo "WHEEL_USERS ALL=(ALL:ALL) NOPASSWD: ALL" >> /etc/sudoers
echo "Port 2026" >> /etc/openssh/sshd_config
echo "AllowUsers sshuser" >> /etc/openssh/sshd_config
echo "MaxAuthTries 2" >> /etc/openssh/sshd_config
echo "PasswordAuthentication yes" >> /etc/openssh/sshd_config
echo "Banner /etc/openssh/banner" >> /etc/openssh/sshd_config
echo "Authorized access only" >> /etc/openssh/banner
systemctl restart sshd
```
**ISP**
```
apt-get install chrony -y 
echo -e "server 127.0.0.1 iburst prefer\nhwtimestamp *\nlocal stratum 5\nallow 0/0\ndriftfile /var/lib/chrony/drifr\nntsdumpdir /var/log/chrony" > /etc/chrony.conf
systemctl enable --now chronyd 
systemctl restart chronyd 
chronyc sources
chronyc tracking | grep Stratum  
```
**HQ-RTR**
```
en
conf t
ntp server 172.16.1.1
ip nat source static tcp 192.168.1.10 2026 172.16.1.4  2026 
ip nat source static tcp 192.168.1.10 80 172.16.1.4 8080
ex
wr mem
```
**BR-RTR**
```
en
conf t
ntp server 172.16.2.1
ip nat source static tcp 192.168.3.10 8080 172.16.2.5 8080
ip nat source static tcp 192.168.3.10 2026 172.16.2.5 2026 
ex
wr mem
```





