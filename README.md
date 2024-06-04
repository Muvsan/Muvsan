<h1>Предварительная настройка</h1>

<h1>Модуль 1</h1>
<h1>Задание 1. Базовая настройка</h1>
<h1>Настройка BR-R</h1>
hostnamectl set-hostname BR-R
newgrp

BR:
10.100.70.1/28
2002:aa:aa:10::1/124

BR-ISP:
20.20.20.2/24 - 20.20.20.1  
1001:aa:aa:20::2/64 - 1001:aa:aa:20::1

<h1>Настройка BR-SRV</h1>
hostnamectl set-hostname BR-SRV
newgrp

BR:
10.100.70.2/28 - 10.100.70.1
2002:aa:aa:10::2/124 - 2002:aa:aa:10::1

<h1>Настройка CLI</h1>
hostnamectl set-hostname BR-R
newgrp

ISP:
30.30.30.2/24 - 30.30.30.1	
1001:aa:aa:30::2/64 - 1001:aa:aa:30::1

<h1>HQ-R</h1>
hostnamectl set-hostname HQ-R
newgrp

HQ:
192.168.100.1/26
2002:aa:aa:192::1/122

HQ-ISP:
10.10.10.2/24 - 10.10.10.1	
1001:aa:aa:10::2/64 - 1001:aa:aa:10::1

<h1>Настройка HQ-SRV</h1>
hostnamectl set-hostname HQ-SRV
newgrp

HQ:
192.168.100.2/26 - 192.168.100.1
2002:aa:aa:192::2/122 - 2002:aa:aa:192::1

<h1>Настройка ISP</h1>
hostnamectl set-hostname ISP
newgrp

HQ-ISP:
10.10.10.1/24
1001:aa:aa:10::1/64

ISP:
30.30.30.1/24
1001:aa:aa:30::1/64

BR-ISP:
20.20.20.1/24
1001:aa:aa:20::1/64

<h1>Задание 2. Настройка FRR</h1>
<h1>Настройка BR-R</h1>
systemctl restart networking
apt install frr
nano /etc/frr/daemons (ospfd ospf6d - yes)
nano /etc/sysctl.conf (Раскоментить ipv4/ipv6 forwarding)
sysctl -p 
systemctl enable frr
systemctl restart frr

Отключить VM Network
systemctl restart networking
vtysh 
conf t
    router ospf (указываем все сети к роутеру, с ipv4)
        network 20.20.20.0/24 area 0
        network 10.100.70.0/28 area 0
    router ospf6 (указываем все сети к роутеру, с ipv6)
        area 0.0.0.0 range 1001:aa:aa:20::0/64
        area 0.0.0.0 range 2002:aa:aa:10::0/124
ip a (узнаем название физ. интерфейсов (ens 161/224/256 и тп)
int ens161 
    ipv6 ospf6 area 0.0.0.0
int ensXXX 
    ipv6 ospf6 area 0.0.0.0	
int ensXXX 
    ipv6 ospf6 area 0.0.0.0
write

<h1>Настройка HQ-R</h1>
systemctl restart networking
apt install frr
nano /etc/frr/daemons (ospfd ospf6d - yes)
nano /etc/sysctl.conf (Раскоментить ipv4/ipv6 forwarding)
sysctl -p 
systemctl enable frr
systemctl restart frr

Отключить VM Network
systemctl restart networking
vtysh 
conf t
    router ospf (указываем все сети к роутеру, с ipv4)
        network 10.10.10.0/24 area 0
        network 192.168.100.0/26 area 0
    router ospf6 (указываем все сети к роутеру, с ipv6)
        area 0.0.0.0 range 1001:aa:aa:10::0/64
        area 0.0.0.0 range 2002:aa:aa:192::0/122
ip a (узнаем название физ. интерфейсов (ens 161/224/256 и тп)
int ens161 
    ipv6 ospf6 area 0.0.0.0
int ensXXX 
    ipv6 ospf6 area 0.0.0.0	
int ensXXX 
    ipv6 ospf6 area 0.0.0.0
write

<h1>Настройка ISP</h1>
systemctl restart networking
apt install frr
nano /etc/frr/daemons (ospfd ospf6d - yes)
nano /etc/sysctl.conf (Раскоментить ipv4/ipv6 forwarding)
sysctl -p 
systemctl enable frr
systemctl restart frr

Отключить VM Network
systemctl restart networking
vtysh 
conf t
    router ospf (указываем все сети к роутеру, с ipv4)
        network 10.10.10.0/24 area 0
		network 20.20.20.0/24 area 0
		network 30.30.30.0/24 area 0
    router ospf6 (указываем все сети к роутеру, с ipv6)
        area 0.0.0.0 range 1001:aa:aa:10::0/64
		area 0.0.0.0 range 1001:aa:aa:20::0/64
		area 0.0.0.0 range 1001:aa:aa:30::0/64
ip a (узнаем название физ. интерфейсов (ens 161/224/256 и тп)
int ens161 
    ipv6 ospf6 area 0.0.0.0
int ensXXX 
    ipv6 ospf6 area 0.0.0.0	
int ensXXX 
    ipv6 ospf6 area 0.0.0.0
write

<h1>Задание 3. DHCP</h1>
<h1>Нсатройка HQ-R</h1>
apt install isc-dhcp-server

nano /etc/default/isc-dhcp-server
INTERFACESv4="ens224"

rm /etc/dhcp/dhcpd.conf, 
nano /etc/dhcp/dhcpd.conf, 

subnet 192.168.100.0 netmask 255.255.255.192 {
range 192.168.100.3 192.168.100.63;
option routers 192.168.100.1;
}

service isc-dhcp-server start/stop/restart (управление службой dhcp)

<h1>Задание 4. Создание пользователей</h1>
<h1>Настройка BR-R</h1>
Отключить VM Network
systemctl restart networking
adduser branch_admin
adduser network_admin

visudo
#User privilege specification
root    ALL=(ALL:ALL) allow
branch_admin    ALL=(ALL:ALL) ALL
network_admin   ALL=(ALL:ALL) ALL
  
<h1>Настройка BR-SRV</h1>
Отключить VM Network
systemctl restart networking
adduser branch_admin
adduser network_admin

visudo
#User privilege specification
root    ALL=(ALL:ALL) allow
branch_admin    ALL=(ALL:ALL) ALL
network_admin   ALL=(ALL:ALL) ALL

<h1>Настройка CLI</h1>
Отключить VM Network
systemctl restart networking
adduser admin

visudo
#User privilege specification
root    ALL=(ALL:ALL) allow
admin    ALL=(ALL:ALL) ALL

<h1>Настройка HQ-R</h1>
Отключить VM Network
systemctl restart networking
adduser admin
adduser network_admin

visudo
#User privilege specification
root    ALL=(ALL:ALL) allow
admin    ALL=(ALL:ALL) ALL
network_admin   ALL=(ALL:ALL) ALL

<h1>Настройка HQ-SRV</h1>
Отключить VM Network
systemctl restart networking
adduser admin

visudo
#User privilege specification
root    ALL=(ALL:ALL) allow
admin    ALL=(ALL:ALL) ALL

<h1>Настройка ISP</h1>
Отключить VM Network
systemctl restart networking
adduser admin
adduser network_admin

visudo
#User privilege specification
root    ALL=(ALL:ALL) allow
admin    ALL=(ALL:ALL) ALL
network_admin   ALL=(ALL:ALL) ALL

<h1>Задание 5. Измерить пропускнуб способность</h1>
<h1>Настройка HQ-R</h1>
systemctl restart networking
apt install iperf3
iperf3 -c (ip адрес проверяемой машины) -i1 -t20

<h1>Настройка ISP</h1>
systemctl restart networking
apt install iperf3
iperf3 -c (ip адрес проверяемой машины) -i1 -t20

<h1>Задание 6. Бэкап</h1>
<h1>Настройка HQ-R</h1>
mkdir /mnt/backup
touch /etc/backup.sh
nano /etc/etc/backup.sh

#1/bin/bash
backup_files="/home /etc /root /boot /opt"
dest="/mnt/backup"
archive_file="backup.tgz"
tar czf $dest/$archive_file $backup_files
ls -lh $dest

bash /etc/backup.sh

tar -xvpzf /mnt/backup/backup.tgz -C / --numeric-owner

<h1>Настройка BR-R</h1>
mkdir /mnt/backup
touch /etc/backup.sh
nano /etc/etc/backup.sh

#1/bin/bash
backup_files="/home /etc /root /boot /opt"
dest="/mnt/backup"
archive_file="backup.tgz"
tar czf $dest/$archive_file $backup_files
ls -lh $dest

bash /etc/backup.sh

tar -xvpzf /mnt/backup/backup.tgz -C / --numeric-owner
