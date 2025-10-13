**не работает nginx**
(ansible) черновик
```
ssh-keygen -f id_rsa -t rsa -N ''
ssh-copy-id -p 2026 -i ~/id_rsa sshuser@192.168.1.10
ssh-copy-id -p 2026 -i ~/id_rsa sshuser@192.168.2.10
```

**BR-SRV**

**Перейти в PVE > выбрать пункт local (AltPVE) > ISO Images > Upload > 
Выбрать путь к ISO > Upload;**
**Отключить ВМ HQ-SRV / BR-SRV > в HQ-SRV / BR-SRV выбрать пункт Hardware > 
Add > CD/DVD Drive > Выбрать Storage (local) и ISO Image (Additional.iso) > Add**

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

mount -o loop /dev/sr0
apt-get update && apt-get install docker-compose docker-engine -y
systemctl enable --now docker
mount -o loop /dev/sr0
docker load < /media/ALTLinux/docker/site_latest.tar
docker load < /media/ALTLinux/docker/mariadb_latest.tar
docker images
echo "services:" >> /root/site.yml
echo "  db:" >> /root/site.yml
echo "    image: mariadb" >> /root/site.yml
echo "    container_name: db" >> /root/site.yml
echo "    environment:" >> /root/site.yml
echo "      DB_NAME: testdb" >> /root/site.yml
echo "      DB_USER: test" >> /root/site.yml
echo "      DB_PASS: Passw0rd" >> /root/site.yml
echo "      MYSQL_ROOT_PASSWORD: Passw0rd" >> /root/site.yml
echo "      MYSQL_DATABASE: testdb" >> /root/site.yml
echo "      MYSQL_USER: test" >> /root/site.yml
echo "      MYSQL_PASSWORD: Passw0rd" >> /root/site.yml
echo "    volumes:" >> /root/site.yml
echo "      - db_data:/var/lib/mysql" >> /root/site.yml
echo "    networks:" >> /root/site.yml
echo "      - app_network" >> /root/site.yml
echo "    restart: unless-stopped" >> /root/site.yml
echo "  testapp:" >> /root/site.yml
echo "    image: site" >> /root/site.yml
echo "    container_name: testapp" >> /root/site.yml
echo "    environment:" >> /root/site.yml
echo "      DB_TYPE: maria" >> /root/site.yml
echo "      DB_HOST: db" >> /root/site.yml
echo "      DB_NAME: testdb" >> /root/site.yml
echo "      DB_USER: test" >> /root/site.yml
echo "      DB_PASS: Passw0rd" >> /root/site.yml
echo "      DB_PORT: 3306" >> /root/site.yml
echo "    ports:" >> /root/site.yml
echo "      - "8080:8000"" >> /root/site.yml
echo "    networks:" >> /root/site.yml
echo "      - app_network" >> /root/site.yml
echo "    depends_on:" >> /root/site.yml
echo "      - db" >> /root/site.yml
echo "    restart: unless-stopped" >> /root/site.yml
echo "volumes:" >> /root/site.yml
echo "  db_data:" >> /root/site.yml
echo "networks:" >> /root/site.yml
echo "  app_network:" >> /root/site.yml
echo "    driver: bridge" >> /root/site.yml
docker compose -f site.yml up -d
docker exec -it db mysql -u root -pPassw0rd -e "CREATE DATABASE testdb; CREATE USER 'test'@'%' IDENTIFIED BY 'Passw0rd'; GRANT ALL PRIVILEGES ON testdb.* TO 'test'@'%'; FLUSH PRIVILEGES;"
docker compose -f site.yml down && docker compose -f site.yml up -d
mkdir -p /root/config
echo "docker compose -f /root/site.yml down" >> /root/config/autorestart.sh
echo "systemctl restart docker" >> /root/config/autorestart.sh
echo "docker compose -f /root/site.yml up -d" >> /root/config/autorestart.sh
export EDITOR=vim

```
crontab -e
# Нажимайте i, и в самом конце пишите
@reboot /root/config/autorestart.sh
# Затем нажимайте ESC > :wq


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
apt-get update
apt-get install -y apache2 php8.2 apache2-mod_php8.2 mariadb-server php8.2-{opcache,curl,gd,intl,mysqli,xml,xmlrpc,ldap,zip,soap,mbstring,json,xmlreader,fileinfo,sodium}
mount -o loop /dev/sr0
systemctl enable --now httpd2 mysqld
mysql_secure_installation << EOF
y
y
P@ssw0rd
P@ssw0rd
y
y
y
y
EOF
ex
mariadb -u root -p
CREATE DATABASE webdb;
CREATE USER 'webc'@'localhost' IDENTIFIED BY 'P@ssw0rd';
GRANT ALL PRIVILEGES ON webdb.* TO 'webc'@'localhost';
FLUSH PRIVILEGES;
mysql -e "CREATE DATABASE webdb;"
mysql -e "CREATE USER 'webc'@'localhost' IDENTIFIED BY 'P@ssw0rd';"
mysql -e "GRANT ALL PRIVILEGES ON webdb.* TO 'webc'@'localhost';"
mysql -e "FLUSH PRIVILEGES;"
iconv -f UTF-16LE -t UTF-8 /media/ALTLinux/web/dump.sql > /tmp/dump_utf8.sql
mysql -u root webdb < /tmp/dump_utf8.sql
chmod 777 /var/www/html
cp /media/ALTLinux/web/index.php /var/www/html
cp /media/ALTLinux/web/logo.png /var/www/html
rm -f /var/www/html/index.html
chown apache2:apache2 /var/www/html
systemctl restart httpd2
echo "\n$servername = "localhost";\n$username = "webc;\n$password = "P@ssw0rd";\n$dbname = "webdb";" >> /var/www/html/index.php
systemctl restart httpd2
sed -i "s/\$servername = .*;/\$servername = 'localhost';/" /var/www/html/index.php
sed -i "s/\$dbname = .*;/\$dbname = 'webdb';/" /var/www/html/index.php
sed -i "s/\$password = .*;/\$password = 'P@ssw0rd';/" /var/www/html/index.php
sed -i "s/\$username = .*;/\$username = 'webc';/" /var/www/html/index.php

```
**HQ-CLI**
```
systemctl restart network
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
systemctl restart network
apt-get update && apt-get install yandex-browser-stable -y
```
**ISP**
```
apt-get install chrony -y 
echo -e "server 127.0.0.1 iburst prefer\nhwtimestamp *\nlocal stratum 5\nallow 0/0\ndriftfile /var/lib/chrony/drifr\nntsdumpdir /var/log/chrony" > /etc/chrony.conf
systemctl enable --now chronyd 
systemctl restart chronyd 
chronyc sources
chronyc tracking | grep Stratum
apt-get install nginx -y
apt-get install apache2-htpasswd -y
htpasswd -bc /etc/nginx/.htpasswd WEB P@ssw0rd
echo "server {" >> /etc/nginx/sites-available.d/proxy.conf
echo "    listen 80;" >> /etc/nginx/sites-available.d/proxy.conf
echo "    server_name web.au-team.irpo;" >> /etc/nginx/sites-available.d/proxy.conf
echo "    auth_basic "Restricted Access";" >> /etc/nginx/sites-available.d/proxy.conf
echo "    auth_basic_user_file /etc/nginx/.htpasswd;" >> /etc/nginx/sites-available.d/proxy.conf
echo "    location / {" >> /etc/nginx/sites-available.d/proxy.conf
echo "        proxy_pass http://172.16.1.4:8080;" >> /etc/nginx/sites-available.d/proxy.conf
echo "        proxy_set_header Host \$host;" >> /etc/nginx/sites-available.d/proxy.conf
echo "        proxy_set_header X-Real-IP \$remote_addr;" >> /etc/nginx/sites-available.d/proxy.conf
echo "    }" >> /etc/nginx/sites-available.d/proxy.conf
echo "}" >> /etc/nginx/sites-available.d/proxy.conf
echo "server {" >> /etc/nginx/sites-available.d/proxy.conf
echo "    listen 80;" >> /etc/nginx/sites-available.d/proxy.conf
echo "    server_name docker.au-team.irpo;" >> /etc/nginx/sites-available.d/proxy.conf
echo "    location / {" >> /etc/nginx/sites-available.d/proxy.conf
echo "        proxy_pass http://172.16.2.5:8080;" >> /etc/nginx/sites-available.d/proxy.conf
echo "        proxy_set_header Host \$host;" >> /etc/nginx/sites-available.d/proxy.conf
echo "        proxy_set_header X-Real-IP \$remote_addr;" >> /etc/nginx/sites-available.d/proxy.conf
echo "    }" >> /etc/nginx/sites-available.d/proxy.conf
echo "}" >> /etc/nginx/sites-available.d/proxy.conf
ln -s /etc/nginx/sites-available.d/proxy.conf /etc/nginx/sites-enabled.d/
mv /etc/nginx/sites-available.d/default.conf /root/
systemctl enable --now nginx
systemctl restart nginx
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





