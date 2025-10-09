samba
**BR-SRV**

apt-get update && apt-get install task-samba-dc -y
apt-get install sudo-samba-schema -y
apt-get install chrony -y
apt-get install docker-compose docker-engine -y
apt-get install ansible -y 
rm -rf /etc/samba/smb.conf 
echo "nameserver 192.168.1.10" >> /etc/resolv.conf

echo "192.168.3.10     br-srv.au-team.irpo" >> vim /etc/hosts 
samba-tool domain provision --realm=AU-TEAM.IRPO --domain=AU-TEAM --adminpass=P@ssw0rd --dns-backend=SAMBA_INTERNAL --server-role=dc --option='dns forwarder=192.168.1.10'

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




**HQ-SRV**

echo "server=/au-team.irpo/192.168.3.10" >> /etc/dnsmasq.conf 
systemctl restart dnsmasq 
apt-get update && apt-get install fdisk -y

mdadm --create /dev/md0 --level=0 --raid-devices=2 /dev/sd[b-d]
mdadm --detail -scan --verbose > /etc/mdadm.conf


**HQ-CLI**

apt-get update && apt-get install admc -y
system-auth write ad AU-TEAM.IRPO cli AU-TEAM 'administrator' 'P@ssw0rd'


**ANSIBLE**
**HQ-CLI**
```
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
**BR-SRV**
```
apt-get update && apt-get install ansible -y 
echo "VMs:" >> /etc/ansible/hosts 
echo " hosts:" >> /etc/ansible/hosts 
echo "  HQ-SRV:" >> /etc/ansible/hosts 
echo "   ansible_host: 192.168.1.10" >> /etc/ansible/hosts 
echo "   ansible_user: sshuser" >> /etc/ansible/hosts 
echo "   ansible_port: 2026" >> /etc/ansible/hosts 
echo "  HQ-CLI:" >> /etc/ansible/hosts 
echo "   ansible_host: 192.168.2.10" >> /etc/ansible/hosts 
echo "   ansible_user: sshuser" >> /etc/ansible/hosts 
echo "   ansible_port: 2026" >> /etc/ansible/hosts 
echo "  HQ-RTR:" >> /etc/ansible/hosts 
echo "   ansible_host: 192.168.1.1" >> /etc/ansible/hosts 
echo "   ansible_user: net_admin" >> /etc/ansible/hosts 
echo "   ansible_password: P@ssw0rd" >> /etc/ansible/hosts 
echo "   ansible_connection: network_cli" >> /etc/ansible/hosts
echo "   ansible_network_os: ios" >> /etc/ansible/hosts
echo "  BR-RTR:" >> /etc/ansible/hosts
echo "   ansible_host: 192.168.3.1" >> /etc/ansible/hosts
echo "   ansible_user: net_admin" >> /etc/ansible/hosts
echo "   ansible_password: P@ssw0rd" >> /etc/ansible/hosts
echo "   ansible_connection: network_cli" >> /etc/ansible/hosts
echo "   ansible_network_os: ios" >> /etc/ansible/hosts
echo "[defaults]" >> /etc/ansible/ansible.cfg
echo "ansible_python_interpreter=/usr/bin/python3" >> /etc/ansible/ansible.cfg
echo "interpreter_python=auto_silent" >> /etc/ansible/ansible.cfg
echo "ansible_host_key_checking=false" >> /etc/ansible/ansible.cfg

ssh-keygen -f id_rsa -t rsa -N ''
ssh-copy-id -p 2026 sshuser@192.168.1.10
ssh-copy-id -p 2026 sshuser@192.168.2.10
```






