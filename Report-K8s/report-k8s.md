https://graspingtech.com/vcenter-create-ubuntu-vm/
created on the ESXi host, 10.1.0.110-114 using the vsphere client at 10.1.0.9
administrator@vsphere.local
Bu$ch3@dm!n

Nodes:
reports
reports2
reports3

Hostnames:
moto.BUSCHE-CNC.com
sudo hostnamectl set-hostname reports.BUSCHE-CNC.com
sudo hostnamectl set-hostname reports2.BUSCHE-CNC.com
sudo hostnamectl set-hostname reports3.BUSCHE-CNC.com

frt-ubu
manual
address: 172.20.1.190
netmask: 255.255.254.0
gateway: 172.20.0.1
automatic: 172.20.0.39, 10.30.1.27

moto 
automatic

Avi-ubu
manual
address:172.20.88.16,172.20.88.17,172.20.88.18,172.20.88.19
netmask: 255.255.252.0
gateway: 172.20.88.1

IP Address
10.1.0.110
10.1.0.111
10.1.0.112
10.1.0.113
10.1.0.114
10.1.0.115 Moxa is using
115

DNS: 10.1.2.69 10.1.2.70 172.20.0.39
Netmask:255.255.252.0
Gateway: 10.1.1.205


Notes:
10.1.0.9
administrator@vsphere.local
Bu$ch3@dm!n

FRT-UBU 
172.20.1.190
AVI-UBU
Moto - 10.1.1.83

Mobex: 
Netmask:255.255.252.0
Gateway:10.1.1.205 
Dns: 10.1.2.69, 172.20.0.39 
Dns suffix: BUSCHE-CNC.COM 
