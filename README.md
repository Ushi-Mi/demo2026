**Монтирование образа.**

Зайти по айпи адресу, выдаваемый машиной, затем зайти в его хранилище > ISO Образы
<img width="841" height="335" alt="image" src="https://github.com/user-attachments/assets/57254d6d-2fa6-42fa-91bb-87079de325ea" />

Затем нужно загрузить образ, который требуется (В данном случае Additional.iso)
<img width="421" height="309" alt="image" src="https://github.com/user-attachments/assets/070394ec-5e25-46ff-8bc4-46b4326714ce" />
Затем нужно вмонтировать этот образ в HQ-SRV и BR-SRV: Нужно нажать по конкретной машине -> Hardware, Add > CD|DVD drive -> Дальше всё как на скрине и нажать "ОК"
<img width="417" height="286" alt="image" src="https://github.com/user-attachments/assets/10b8ae3c-595d-4246-a818-c0650515c546" />


Добавление дисков для raid
PVE > HQ-SRV в Hardware > Add > Hard Disk > В пункте Storage выставляем local, размер диска 1 Gb > Добавляем всего 2 диска.
 <img width="616" height="290" alt="image" src="https://github.com/user-attachments/assets/82bcc3a6-e864-4865-852a-a9dcba28551e" />

**HQ-RTR | BR-RTR (Ecorouter)**
Базовая коммутация до ISP-a.
Название пользователя: admin, пароль: admin

**HQ-RTR**
```
en  
conf t
interface int0
description "to isp"
ip address 172.16.1.4/28
exit
port te0
service-instance te0/int0
encapsulation untagged
exit
exit
interface int0
connect port te0 service-instance te0/int0
exit
ip route 0.0.0.0 0.0.0.0 172.16.1.1
no security default
exit
wr
```
BR-RTR
en
conf t
interface int0
description "to isp"
ip address 172.16.2.5/28
exit
port te0
service-instance te0/int0
encapsulation untagged
exit
exit
interface int0
connect port te0 service-instance te0/int0
exit
ip route 0.0.0.0 0.0.0.0 172.16.2.1
no security default
exit
wr
HQ-SRV | HQ-CLI | BR-SRV
Базовая коммутация до роутеров.
Название пользователя: root|user, Пароль: toor|resu (На HQ-SRV|BR-SRV root, на HQ-CLI user)

HQ-SRV
mkdir -p /etc/net/ifaces/ens20
echo -e "DISABLED=no\nTYPE=eth\nBOOTPROTO=static\nCONFIG_IPv4=yes" > /etc/net/ifaces/ens20/options
echo "192.168.1.10/26" > /etc/net/ifaces/ens20/ipv4address
echo "default via 192.168.1.1" > /etc/net/ifaces/ens20/ipv4route
systemctl restart network
HQ-CLI
mkdir -p /etc/net/ifaces/ens20
echo -e "DISABLED=no\nTYPE=eth\nBOOTPROTO=static\nCONFIG_IPv4=yes" > /etc/net/ifaces/ens20/options
echo "192.168.2.10/28" > /etc/net/ifaces/ens20/ipv4address
echo "default via 192.168.2.1" > /etc/net/ifaces/ens20/ipv4route
systemctl restart network
BR-SRV
mkdir -p /etc/net/ifaces/ens20
echo -e "DISABLED=no\nTYPE=eth\nBOOTPROTO=static\nCONFIG_IPv4=yes" > /etc/net/ifaces/ens20/options
echo "192.168.3.10/27" > /etc/net/ifaces/ens20/ipv4address
echo "default via 192.168.3.1" > /etc/net/ifaces/ens20/ipv4route
systemctl restart network
Разрешение на логирование через root(делайте только в случае автоматизации, в реальной жизни никто так делать конечно же не будет, всё сделано в целях автоматизации)
BR-SRV | HQ-SRV
echo -e "PermitRootLogin yes\nPort 2026" >> /etc/openssh/sshd_config
systemctl enable --now sshd
systemctl restart sshd
HQ-CLI
echo -e "PermitRootLogin yes\nPort 2222" >> /etc/openssh/sshd_config
systemctl enable --now sshd
systemctl restart sshd
ISP
Настройка DHCP интерфейса
mkdir -p /etc/net/ifaces/ens20
echo -e "DISABLED=no\nTYPE=eth\nBOOTPROTO=dhcp\nCONFIG_IPv4=yes" > /etc/net/ifaces/ens20/options
echo "net.ipv4.ip_forward = 1" >> /etc/net/sysctl.conf
systemctl restart network
Инструкция для активации ISP-a
apt-get update && apt-get install git -y && git clone https://github.com/NiKeNO1540/DEMO-2025-testing && chmod +x DEMO-2025-testing/Full_Config_Progression_AIO.sh && ./DEMO-2025-testing/Full_Config_Progression_AIO.sh
