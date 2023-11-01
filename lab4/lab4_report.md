University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Introduction in routing](https://github.com/itmo-ict-faculty/introduction-in-routing)

Year: 2022/2023

Group: K33202

Author: Samsonov Stanislav

Lab: Lab4

Date of create: 01.12.2022

Date of finished: 10.12.2022

## Отчёт по лабораторной работе №4 "Эмуляция распределенной корпоративной сети связи, настройка iBGP, организация L3VPN, VPLS"

**Цель работы:** изучить протоколы BGP, MPLS и правила организации L3VPN и VPLS.

**Ход выполнения работы:**

1. Текст файла для развертывания тестовой сети с расширением .yaml

```
name: lab4

mgmt:
  network: statics
  ipv4_subnet: 172.20.20.0/24

topology:
  nodes:
    R01.NY: #10.10.10.1/32
      kind: vr-mikrotik_ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.2

    R01.LND: #10.10.10.2/32
      kind: vr-mikrotik_ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.3

    R01.HKI: #10.10.10.3/32
      kind: vr-mikrotik_ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.4

    R01.SPB: #10.10.10.4/32
      kind: vr-mikrotik_ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.5

    R01.LBN: #10.10.10.5/32
      kind: vr-mikrotik_ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.6

    R01.SVL: #10.10.10.6/32
      kind: vr-mikrotik_ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.7

    PC1:
      kind: vr-mikrotik_ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.8

    PC2:
      kind: vr-mikrotik_ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.9

    PC3:
      kind: vr-mikrotik_ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.10
  

  links: 
    - endpoints: ["R01.NY:eth4","PC2:eth4"]      #192.168.20.10/24 - 192.168.20.11/24
    - endpoints: ["R01.NY:eth3","R01.LND:eth3"]  #10.10.1.1/30 - 10.10.1.2/30
    - endpoints: ["R01.LND:eth1","R01.HKI:eth2"] #10.10.2.1/30 - 10.10.2.2/30   
    - endpoints: ["R01.LND:eth2","R01.LBN:eth1"] #10.10.4.1/30 - 10.10.4.2/30
    - endpoints: ["R01.LBN:eth2","R01.HKI:eth1"] #10.10.5.2/30 - 10.10.5.1/30
    - endpoints: ["R01.LBN:eth3","R01.SVL:eth3"] #10.10.6.1/30
    - endpoints: ["R01.SVL:eth4","PC3:eth4"]     #192.168.30.10/24 - 192.168.30.11/24
    - endpoints: ["R01.HKI:eth3","R01.SPB:eth3"] #10.10.3.1/30 - 10.10.3.2/30
    - endpoints: ["R01.SPB:eth4","PC1:eth4"]     #192.168.10.10/24 - 192.168.10.11/24
```
2. Схема связи

![](https://github.com/Slabhide/2022_2023-introduction_in_routing-k33202-samsonov_stanislav/blob/main/lab4/draw.io.png "Схема сети")

3. Тексты конфигураций для сетевых устройств

* Роутер R01.NY

```
# software id = 
#
#
#
/interface bridge
add name=Lo0
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default redistribute-connected=yes router-id=10.10.10.1
/routing ospf instance
set [ find default=yes ] name=ospf0 router-id=10.10.10.1
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=192.168.20.10/24 interface=ether2 network=192.168.20.0
add address=10.10.1.1/30 interface=ether4 network=10.10.1.0
add address=10.10.10.1 interface=Lo0 network=10.10.10.1
/ip dhcp-client
add disabled=no interface=ether1
add disabled=no interface=ether2
/ip route vrf
add export-route-targets=65530:100 import-route-targets=65530:100 interfaces=ether2 route-distinguisher=65530:100 \
    routing-mark=VRF_DEVOPS
/mpls ldp
set enabled=yes transport-address=10.10.10.1
/mpls ldp interface
add interface=ether4
/routing bgp instance vrf
add redistribute-connected=yes routing-mark=VRF_DEVOPS
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=10.10.10.2 remote-as=65530 update-source=Lo0
/routing ospf network
add area=backbone
/system identity
set name=R01.NY
```

* Роутер R01.LND

```
# software id = 
#
#
#
/interface bridge
add name=Lo0
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default router-id=10.10.10.2
/routing ospf instance
set [ find default=yes ] name=ospf0 router-id=10.10.10.2
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.1.2/30 interface=ether4 network=10.10.1.0
add address=10.10.2.1/30 interface=ether2 network=10.10.2.0
add address=10.10.4.1/30 interface=ether3 network=10.10.4.0
add address=10.10.10.2 interface=Lo0 network=10.10.10.2
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes transport-address=10.10.10.2
/mpls ldp interface
add interface=ether2
add interface=ether3
add interface=ether4
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer3 remote-address=10.10.10.1 remote-as=65530 update-source=Lo0
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=10.10.10.3 remote-as=65530 route-reflect=yes \
    update-source=Lo0
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer2 remote-address=10.10.10.5 remote-as=65530 route-reflect=yes \
    update-source=Lo0
/routing ospf network
add area=backbone
/system identity
set name=R01.LND
```

* Роутер R01.HKI

```
# software id = 
#
#
#
/interface bridge
add name=Lo0
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default router-id=10.10.10.3
/routing ospf instance
set [ find default=yes ] name=ospf0 router-id=10.10.10.3
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.10.3 interface=Lo0 network=10.10.10.3
add address=10.10.2.2/30 interface=ether3 network=10.10.2.0
add address=10.10.5.1/30 interface=ether2 network=10.10.5.0
add address=10.10.3.1/30 interface=ether4 network=10.10.3.0
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp interface
add interface=ether2
add interface=ether3
add interface=ether4
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=10.10.10.2 remote-as=65530 route-reflect=yes \
    update-source=Lo0
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer2 remote-address=10.10.10.5 remote-as=65530 route-reflect=yes \
    update-source=Lo0
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer3 remote-address=10.10.10.4 remote-as=65530 update-source=Lo0
/routing ospf network
add area=backbone
/system identity
set name=R01.HKI
```

* Роутер R01.LBN

```
# software id = 
#
#
#
/interface bridge
add name=Lo0
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default router-id=10.10.10.5
/routing ospf instance
set [ find default=yes ] name=ospf0 router-id=10.10.10.5
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.10.5 interface=Lo0 network=10.10.10.5
add address=10.10.4.2/30 interface=ether2 network=10.10.4.0
add address=10.10.5.2/30 interface=ether3 network=10.10.5.0
add address=10.10.6.1/30 interface=ether4 network=10.10.6.0
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes transport-address=10.10.10.5
/mpls ldp interface
add interface=ether2
add interface=ether3
add interface=ether4
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=10.10.10.2 remote-as=65530 route-reflect=yes update-source=Lo0
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer2 remote-address=10.10.10.3 remote-as=65530 route-reflect=yes update-source=Lo0
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer3 remote-address=10.10.10.6 remote-as=65530 update-source=Lo0
/routing ospf network
add area=backbone
/system identity
set name=R01.LBN
```

* Роутер R01.SPB

```
# software id = 
#
#
#
/interface bridge
add name=Lo0
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default router-id=10.10.10.4
/routing ospf instance
set [ find default=yes ] name=ospf0 router-id=10.10.10.4
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.3.2/30 interface=ether4 network=10.10.3.0
add address=10.10.10.4 interface=Lo0 network=10.10.10.4
add address=192.168.10.10/24 interface=ether2 network=192.168.10.0
/ip dhcp-client
add disabled=no interface=ether1
/ip route vrf
add export-route-targets=65530:100 import-route-targets=65530:100 interfaces=ether2 route-distinguisher=65530:100 routing-mark=VRF_DEVOPS
/mpls ldp
set enabled=yes transport-address=10.10.10.4
/mpls ldp interface
add interface=ether4
/routing bgp instance vrf
add redistribute-connected=yes routing-mark=VRF_DEVOPS
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=10.10.10.3 remote-as=65530 update-source=Lo0
/routing ospf network
add area=backbone
/system identity
set name=R01.SPB
```

* Роутер R01.SVL

```
# software id = 
#
#
#
/interface bridge
add name=Lo0
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default router-id=10.10.10.6
/routing ospf instance
set [ find default=yes ] name=ospf0 router-id=10.10.10.6
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.10.6 interface=Lo0 network=10.10.10.6
add address=192.168.30.10/24 interface=ether2 network=192.168.30.0
add address=10.10.6.2/30 interface=ether4 network=10.10.6.0
/ip dhcp-client
add disabled=no interface=ether1
/ip route vrf
add export-route-targets=65530:100 import-route-targets=65530:100 interfaces=ether2 route-distinguisher=65530:100 routing-mark=VRF_DEVOPS
/mpls ldp
set enabled=yes transport-address=10.10.10.6
/mpls ldp interface
add interface=ether4
/routing bgp instance vrf
add redistribute-connected=yes routing-mark=VRF_DEVOPS
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=10.10.10.5 remote-as=65530 update-source=Lo0
/routing ospf network
add area=backbone
/system identity
set name=R01.SVL
```

4. Проверка связности между VRF

![](https://github.com/Slabhide/2022_2023-introduction_in_routing-k33202-samsonov_stanislav/blob/main/lab4/VRF.png "VRF")

5. Настройка VPLS (конфигурации устройств)

* PC1

```
# software id = 
#
#
#
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=192.168.0.1/24 interface=ether2 network=192.168.0.0
/ip dhcp-client
add disabled=no interface=ether1
/system identity
set name=PC1
```

* PC2

```
# software id = 
#
#
#
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=192.168.0.2/24 interface=ether2 network=192.168.0.0
/ip dhcp-client
add disabled=no interface=ether1
/system identity
set name=PC2
```

* PC3

```
# software id = 
#
#
#
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=192.168.0.3/24 interface=ether2 network=192.168.0.0
/ip dhcp-client
add disabled=no interface=ether1
/system identity
set name=PC3
```

* R01.NY

```
# software id = 
#
#
#
/interface bridge
add name=Lo
add name=VPLS
/interface vpls
add disabled=no l2mtu=1500 mac-address=02:3B:11:73:24:F9 name=VPLS3 remote-peer=10.10.10.6 vpls-id=10:0
add disabled=no l2mtu=1500 mac-address=02:CB:23:D2:D0:7C name=vpls1 remote-peer=10.10.10.4 vpls-id=10:0
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default router-id=10.10.10.1
/routing ospf instance
set [ find default=yes ] router-id=10.10.10.1
/interface bridge port
add bridge=VPLS interface=ether2
add bridge=VPLS interface=vpls1
add bridge=VPLS interface=VPLS3
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.10.1 interface=Lo network=10.10.10.1
add address=10.10.1.1/30 interface=ether4 network=10.10.1.0
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes
/mpls ldp interface
add interface=ether4
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=10.10.10.2 remote-as=65530 update-source=Lo
/routing ospf network
add area=backbone
/system identity
set name=R01.NY
```

* R01.SPB

```
# software id = 
#
#
#
/interface bridge
add name=Lo
add name=VPLS
/interface vpls
add disabled=no l2mtu=1500 mac-address=02:5E:29:75:91:55 name=vpls1 remote-peer=10.10.10.1 vpls-id=10:0
add disabled=no l2mtu=1500 mac-address=02:B5:27:55:42:F5 name=vpls2 remote-peer=10.10.10.6 vpls-id=10:0
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default router-id=10.10.10.4
/routing ospf instance
set [ find default=yes ] router-id=10.10.10.4
/interface bridge port
add bridge=VPLS interface=ether2
add bridge=VPLS interface=vpls1
add bridge=VPLS interface=vpls2
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.3.2/30 interface=ether4 network=10.10.3.0
add address=10.10.10.4 interface=Lo network=10.10.10.4
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes
/mpls ldp interface
add interface=ether4
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=10.10.10.3 remote-as=65530 update-source=Lo
/routing ospf network
add area=backbone
/system identity
set name=R01.SPB
```

* R01.SVL

```
# software id = 
#
#
#
/interface bridge
add name=Lo
add name=VPLS
/interface vpls
add disabled=no l2mtu=1500 mac-address=02:3F:4F:5F:C7:41 name=VPLS3 remote-peer=10.10.10.1 vpls-id=10:0
add disabled=no l2mtu=1500 mac-address=02:C0:D7:89:7A:9C name=vpls2 remote-peer=10.10.10.4 vpls-id=10:0
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default router-id=10.10.10.6
/routing ospf instance
set [ find default=yes ] router-id=10.10.10.6
/interface bridge port
add bridge=VPLS interface=ether2
add bridge=VPLS interface=VPLS3
add bridge=VPLS interface=vpls2
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.6.2/30 interface=ether4 network=10.10.6.0
add address=10.10.10.6 interface=Lo network=10.10.10.6
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes
/mpls ldp interface
add interface=ether4
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=10.10.10.5 remote-as=65530 update-source=Lo
/routing ospf network
add area=backbone
/system identity
set name=R01.SVL
```
6. Проверка связности ПК.

![](https://github.com/Slabhide/2022_2023-introduction_in_routing-k33202-samsonov_stanislav/blob/main/lab4/Ping_1.png "Пинг1")
![](https://github.com/Slabhide/2022_2023-introduction_in_routing-k33202-samsonov_stanislav/blob/main/lab4/Ping_2.png "Пинг2")
![](https://github.com/Slabhide/2022_2023-introduction_in_routing-k33202-samsonov_stanislav/blob/main/lab4/Ping_3.png "Пинг3")

