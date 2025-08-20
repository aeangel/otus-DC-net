### Задание Underlay. OSPF

Цель:
Настроить OSPF для Underlay сети.

Задачи:
Настроите OSPF в Underlay сети, для IP связанности между всеми сетевыми устройствами.
Зафиксируете в документации - план работы, адресное пространство, схему сети, конфигурацию устройств
Убедитесь в наличии IP связанности между устройствами в OSFP домене


### Схема стенда

![stand-plan](stand-plan.png)

Стенд делаем по принципу - хосты linux, leaf - frr, spine - arista

### Распределение адресного пространства для Underlay

План составлен с учетом 10.x.y.z, где x - номер DC, y - номер spine, z - по очереди для подключения leaf
Адреса для хостов - 172.16.x.z/24, где x - номер leaf, z - по порядку адрес хоста, на leaf ip .1
Адреса loopback 192.168.a.b/32, где a - 1 для spine, 2 - для leaf, b - номер spine, leaf по порядку
Адресацию ipv6 делаем по прицнипу из fd00::[IPv4]

Interconnect ipv4 ipv6

| Device A | Interface A | IPv4 A        | IPv6 A               | Device B | Interface B | IPv4 B        | IPv6 B               |
|----------|-------------|---------------|----------------------|----------|-------------|---------------|----------------------|
| Spine-1  | Eth1        | 10.1.1.0/31    | fd00::10:1:1:0/127    | Leaf-1   | Eth8        | 10.1.1.1/31    | fd00::10:1:1:1/127    |
| Spine-1  | Eth2        | 10.1.1.2/31    | fd00::10:1:1:2/127    | Leaf-2   | Eth8        | 10.1.1.3/31    | fd00::10:1:1:3/127    |
| Spine-1  | Eth3        | 10.1.1.4/31    | fd00::10:1:1:4/127    | Leaf-3   | Eth8        | 10.1.1.5/31    | fd00::10:1:1:5/127    |
| Spine-2  | Eth2        | 10.1.2.0/31    | fd00::10:2:1:0/127    | Leaf-1   | Eth9        | 10.1.2.1/31    | fd00::10:2:1:1/127    |
| Spine-2  | Eth2        | 10.1.2.2/31    | fd00::10:2:1:2/127    | Leaf-2   | Eth9        | 10.1.2.3/31    | fd00::10:2:1:3/127    |
| Spine-2  | Eth3        | 10.1.2.4/31    | fd00::10:2:1:4/127    | Leaf-3   | Eth9        | 10.1.2.5/31    | fd00::10:2:1:5/127    |
| Host-1   | Eth1        | 172.16.1.2/24  | fd00::172:16:1:2/116   | Leaf-1   | Eth1        | 172.16.1.1/24  | fd00::172:16:1:1/116   |
| Host-2   | Eth1        | 172.16.2.2/24  | fd00::172:16:2:2/116   | Leaf-2   | Eth1        | 172.16.2.1/24  | fd00::172:16:2:1/116   |
| Host-3   | Eth1        | 172.16.3.2/24  | fd00::172:16:3:2/116  | Leaf-3   | Eth1        | 172.16.3.1/24  | fd00::172:16:3:1/116   |
| Host-4   | Eth1        | 172.16.4.2/24  | fd00::172:16:4:2/116   | Leaf-3   | Eth2        | 172.16.4.1/24  | fd00::172:16:4:1/116   |

loopback

| Device | Loopback ipv4| loopback ipv6|
|-------------|---------------|-----------|
| Spine-1  | 192.168.1.1 | fd00::192:168:1:1 |
| Spine-2  | 192.168.1.2 | fd00::192:168:1:2 |
| Leaf-1   | 192.168.2.1 | fd00::192:168:2:1 |
| Leaf-2   | 192.168.2.2 | fd00::192:168:2:2 |
| Leaf-3   | 192.168.2.2 | fd00::192:168:2:3 |

### Примечание - так как в явном виде не указано что делать с адресацией на хостовых машинах, а в качестве leaf и spine выступают роутеры, до настройки overlay адреса на хостах распределены из разных подсетей.
Когда поднимем overlay засунем все в одну подсеть чтобы эмулировать l2.

### Запуск лабараторной в среде netlab
Так как у самурая только путь, делаем все в netlab-tools.  
Заметки по использованию netlab:  
Для работы bfd в frr нужно включить его принудительно в образе по умолчанию, т.к. в netlab не заявлено поддержки bfd на оборудовании frr.  
Правим файл /usr/local/lib/python3.10/dist-packages/netsim/templates/provider/clab/frr/daemons.j2 включая демона - bfdd=yes.  
Так же если хотим использовать другие протоколы в frr, без подключения модулей в netlab в этом же файле нужно включить желаемых демонов.  
Заметки по лабе:  
Для ospf включаем шифрование для подключений leaf-spine, пароли spine1, spine2 соответсвенно. Clear-text password используем т.к. netlab не может работать с md5 для frr, но так как аутентификация нам нужна для подстраховки от ошибки, не критично использование открытого пароля.
Также включаем bfd для интерфейсов, таймеры явно получатся завышенными, но моя практика показывает что для виртуальных устройств лучше не использовать значения меньше 500 мс, поэтому не критично.


![конфиг файл](./topology.yml)
или под катом

<details>
  <summary>topology.yml </summary>

  ```yml
---
provider: clab
module: [ ospf ]

nodes:
 s1:
  device: eos
  id: 1
  loopback:
    ipv4: 192.168.1.1/32
    ipv6: fd00::192:168:1:1/128
 s2:
  device: eos
  id: 2
  loopback:
    ipv4: 192.168.1.2/32
    ipv6: fd00::192:168:1:2/128
 l1:
  device: frr
  id: 3
  loopback:
    ipv4: 192.168.2.1/32
    ipv6: fd00::192:168:2:1/128
 l2:
  device: frr
  id: 4
  loopback:
    ipv4: 192.168.2.2/32
    ipv6: fd00::192:168:2:2/128
 l3:
  device: frr
  id: 5
  loopback:
    ipv4: 192.168.2.3/32
    ipv6: fd00::192:168:2:3/128
 h1:
  device: linux
 h2:
  device: linux
 h3:
  device: linux
 h4:
  device: linux

links:
#spine1-leaf1,2,3
  - interfaces:
      - node: s1
        ifname: eth1
        ipv4: 10.1.1.0
        ipv6: fd00::10:1:1:0
        ospf:
          password: 'spine1'
          bfd: true
      - node: l1
        ifname: eth8
        ipv4: 10.1.1.1
        ipv6: fd00::10:1:1:1
        ospf:
          password: 'spine1'
          bfd: true
    prefix:
      ipv4: 10.1.1.0/31
      ipv6: fd00::10:1:1:0/127
  - interfaces:
      - node: s1
        ifname: eth2
        ipv4: 10.1.1.2
        ipv6: fd00::10:1:1:2
        ospf:
          password: 'spine1'
          bfd: true
      - node: l2
        ifname: eth8
        ipv4: 10.1.1.3
        ipv6: fd00::10:1:1:3
        ospf:
          password: 'spine1'
          bfd: true
    prefix:
      ipv4: 10.1.1.2/31
      ipv6: fd00::10:1:1:2/127
  - interfaces:
      - node: s1
        ifname: eth3
        ipv4: 10.1.1.4
        ipv6: fd00::10:1:1:4
        ospf:
          password: 'spine1'
          bfd: true
      - node: l3
        ifname: eth8
        ipv4: 10.1.1.5
        ipv6: fd00::10:1:1:5
        ospf:
          password: 'spine1'
          bfd: true
    prefix:
      ipv4: 10.1.1.4/31
      ipv6: fd00::10:1:1:4/127
#spine2-leaf1,2,3
  - interfaces:
      - node: s2
        ifname: eth1
        ipv4: 10.1.2.0
        ipv6: fd00::10:1:2:0
        ospf:
          password: 'spine2'
          bfd: true
      - node: l1
        ifname: eth9
        ipv4: 10.1.2.1
        ipv6: fd00::10:1:2:1
        ospf:
          password: 'spine2'
          bfd: true
    prefix:
      ipv4: 10.1.2.0/31
      ipv6: fd00::10:1:2:0/127
  - interfaces:
      - node: s2
        ifname: eth2
        ipv4: 10.1.2.2
        ipv6: fd00::10:1:2:2
        ospf:
          password: 'spine2'
          bfd: true
      - node: l2
        ifname: eth9
        ipv4: 10.1.2.3
        ipv6: fd00::10:1:2:3
        ospf:
          password: 'spine2'
          bfd: true
    prefix:
      ipv4: 10.1.2.2/31
      ipv6: fd00::10:1:2:2/127
  - interfaces:
      - node: s2
        ifname: eth3
        ipv4: 10.1.2.4
        ipv6: fd00::10:1:2:4
        ospf:
          password: 'spine2'
          bfd: true
      - node: l3
        ifname: eth9
        ipv4: 10.1.2.5
        ipv6: fd00::10:1:2:5
        ospf:
          password: 'spine2'
          bfd: true
    prefix:
      ipv4: 10.1.2.4/31
      ipv6: fd00::10:1:2:4/127
#host1
  - interfaces:
      - node: h1
        ifname: eth1
        ipv4: 172.16.1.2
        ipv6: fd00::172:16:1:2
      - node: l1
        ifname: eth1
        ipv4: 172.16.1.1
        ipv6: fd00::172:16:1:1
        ospf: false
    prefix:
      ipv4: 172.16.1.0/24
      ipv6: fd00::172:16:1:0/116
#host2
  - interfaces:
      - node: h2
        ifname: eth1
        ipv4: 172.16.2.2
        ipv6: fd00::172:16:2:2
      - node: l2
        ifname: eth1
        ipv4: 172.16.2.1
        ipv6: fd00::172:16:2:1
        ospf: false
    prefix:
      ipv4: 172.16.2.0/24
      ipv6: fd00::172:16:1:0/116
#host3
  - interfaces:
      - node: h3
        ifname: eth1
        ipv4: 172.16.3.2
        ipv6: fd00::172:16:3:2
      - node: l3
        ifname: eth1
        ipv4: 172.16.3.1
        ipv6: fd00::172:16:3:1
        ospf: false
    prefix:
      ipv4: 172.16.3.0/24
      ipv6: fd00::172:16:3:0/116
#host4
  - interfaces:
      - node: h4
        ifname: eth1
        ipv4: 172.16.4.2
        ipv6: fd00::172:16:4:2
      - node: l3
        ifname: eth2
        ipv4: 172.16.4.1
        ipv6: fd00::172:16:4:1
        ospf: false
    prefix:
      ipv4: 172.16.4.0/24
      ipv6: fd00::172:16:4:0/116

```
</details>

## Проверка работы

<details>
  <summary>spine-1</summary>
  
  ```txt  
s1#ping 10.1.1.2
PING 10.1.1.2 (10.1.1.2) 72(100) bytes of data.
80 bytes from 10.1.1.2: icmp_seq=1 ttl=64 time=0.137 ms
80 bytes from 10.1.1.2: icmp_seq=2 ttl=64 time=0.002 ms
80 bytes from 10.1.1.2: icmp_seq=3 ttl=64 time=0.004 ms
80 bytes from 10.1.1.2: icmp_seq=4 ttl=64 time=0.018 ms
80 bytes from 10.1.1.2: icmp_seq=5 ttl=64 time=0.005 ms

--- 10.1.1.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.002/0.033/0.137/0.052 ms, ipg/ewma 0.063/0.083 ms
s1#ping 10.1.1.6
PING 10.1.1.6 (10.1.1.6) 72(100) bytes of data.
80 bytes from 10.1.1.6: icmp_seq=1 ttl=64 time=0.129 ms
80 bytes from 10.1.1.6: icmp_seq=2 ttl=64 time=0.013 ms
80 bytes from 10.1.1.6: icmp_seq=3 ttl=64 time=0.012 ms
80 bytes from 10.1.1.6: icmp_seq=4 ttl=64 time=0.007 ms
80 bytes from 10.1.1.6: icmp_seq=5 ttl=64 time=0.031 ms

--- 10.1.1.6 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.007/0.038/0.129/0.046 ms, ipg/ewma 0.096/0.082 ms
s1#ping 10.1.1.10
PING 10.1.1.10 (10.1.1.10) 72(100) bytes of data.
80 bytes from 10.1.1.10: icmp_seq=1 ttl=64 time=0.131 ms
80 bytes from 10.1.1.10: icmp_seq=2 ttl=64 time=0.013 ms
80 bytes from 10.1.1.10: icmp_seq=3 ttl=64 time=0.014 ms
80 bytes from 10.1.1.10: icmp_seq=4 ttl=64 time=0.008 ms
80 bytes from 10.1.1.10: icmp_seq=5 ttl=64 time=0.006 ms

--- 10.1.1.10 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.006/0.034/0.131/0.048 ms, ipg/ewma 0.074/0.081 ms
s1#show ip ro

VRF: default
Source Codes:
       C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route,
       CL - CBF Leaked Route

Gateway of last resort is not set

 C        10.1.1.0/30
           directly connected, Ethernet1
 C        10.1.1.4/30
           directly connected, Ethernet2
 C        10.1.1.8/30
           directly connected, Ethernet3
 C        192.168.1.1/32
           directly connected, Loopback0

s1#show arp
Address         Age (sec)  Hardware Addr   Interface
10.1.1.2          0:02:09  aac1.abff.0f0e  Ethernet1
10.1.1.6          0:02:02  aac1.ab68.d548  Ethernet2
10.1.1.10         0:01:58  aac1.abf2.ccef  Ethernet3
```
</details>

<details>
  <summary>spine-2 </summary>

  ```txt  
s2#ping 10.1.2.2
PING 10.1.2.2 (10.1.2.2) 72(100) bytes of data.
80 bytes from 10.1.2.2: icmp_seq=1 ttl=64 time=0.128 ms
80 bytes from 10.1.2.2: icmp_seq=2 ttl=64 time=0.016 ms
80 bytes from 10.1.2.2: icmp_seq=3 ttl=64 time=0.008 ms
80 bytes from 10.1.2.2: icmp_seq=4 ttl=64 time=0.000 ms
80 bytes from 10.1.2.2: icmp_seq=5 ttl=64 time=0.023 ms

--- 10.1.2.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.000/0.035/0.128/0.047 ms, ipg/ewma 0.092/0.080 ms
s2#ping 10.1.2.6
PING 10.1.2.6 (10.1.2.6) 72(100) bytes of data.
80 bytes from 10.1.2.6: icmp_seq=1 ttl=64 time=0.147 ms
80 bytes from 10.1.2.6: icmp_seq=2 ttl=64 time=0.013 ms
80 bytes from 10.1.2.6: icmp_seq=3 ttl=64 time=0.012 ms
80 bytes from 10.1.2.6: icmp_seq=4 ttl=64 time=0.011 ms
80 bytes from 10.1.2.6: icmp_seq=5 ttl=64 time=0.010 ms

--- 10.1.2.6 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.010/0.038/0.147/0.054 ms, ipg/ewma 0.081/0.091 ms
s2#ping 10.1.2.10
PING 10.1.2.10 (10.1.2.10) 72(100) bytes of data.
80 bytes from 10.1.2.10: icmp_seq=1 ttl=64 time=0.128 ms
80 bytes from 10.1.2.10: icmp_seq=2 ttl=64 time=0.013 ms
80 bytes from 10.1.2.10: icmp_seq=3 ttl=64 time=0.011 ms
80 bytes from 10.1.2.10: icmp_seq=4 ttl=64 time=0.010 ms
80 bytes from 10.1.2.10: icmp_seq=5 ttl=64 time=0.008 ms

--- 10.1.2.10 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.008/0.034/0.128/0.047 ms, ipg/ewma 0.065/0.079 ms
s2#show ip ro

VRF: default
Source Codes:
       C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route,
       CL - CBF Leaked Route

Gateway of last resort is not set

 C        10.1.2.0/30
           directly connected, Ethernet1
 C        10.1.2.4/30
           directly connected, Ethernet2
 C        10.1.2.8/30
           directly connected, Ethernet3
 C        192.168.1.2/32
           directly connected, Loopback0

s2#show arp
Address         Age (sec)  Hardware Addr   Interface
10.1.2.2          0:00:11  aac1.abdd.eb6a  Ethernet1
10.1.2.6          0:00:07  aac1.ab74.efe2  Ethernet2
10.1.2.10         0:00:05  aac1.abf5.61f3  Ethernet3
```
</details>

