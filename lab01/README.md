# lab01-basic-CLOS

#### Задание

1.Соберете топологию CLOS, как на схеме.

2.Распределите адресное пространство для Underlay сети&#x20;

3.Зафиксируете в документации план работ, адресное пространство, схему сети, настройки (если перенесли на оборудование)

#### Схема стенда

![stand-plan](stand-plan.png)

Стенд делаем по принципу - хосты linux, leaf - frr, spine - arista

#### Распределение адресного пространства для Underlay

План составлен с учетом 10.x.y.z, где x - номер DC, y - номер spine, z - по очереди для подключения leaf Адреса для хостов - 172.16.x.z/24, где x - номер leaf, z - по порядку адрес хоста, на leaf ip .1 Адреса loopback 192.168.a.b/32, где a - 1 для spine, 2 - для leaf, b - номер spine, leaf по порядку

Interconnect table

| Device A | Interface A | IP A          | Device B | Interface B | IP B          |
| -------- | ----------- | ------------- | -------- | ----------- | ------------- |
| Spine-1  | Eth1        | 10.1.1.1/30   | Leaf-1   | Eth8        | 10.1.1.2/30   |
| Spine-1  | Eth2        | 10.1.1.5/30   | Leaf-2   | Eth8        | 10.1.1.6/30   |
| Spine-1  | Eth3        | 10.1.1.9/30   | Leaf-3   | Eth8        | 10.1.1.10/30  |
| Spine-2  | Eth2        | 10.1.2.1/30   | Leaf-1   | Eth9        | 10.1.2.2/30   |
| Spine-2  | Eth2        | 10.1.2.5/30   | Leaf-2   | Eth9        | 10.1.2.6/30   |
| Spine-2  | Eth3        | 10.1.2.9/30   | Leaf-3   | Eth9        | 10.1.2.10/30  |
| Host-1   | Eth1        | 172.16.1.2/24 | Leaf-1   | Eth1        | 172.16.1.1/24 |
| Host-2   | Eth1        | 172.16.2.2/24 | Leaf-2   | Eth1        | 172.16.2.1/24 |
| Host-3   | Eth1        | 172.16.3.2/24 | Leaf-3   | Eth1        | 172.16.3.1/24 |
| Host-4   | Eth1        | 172.16.4.2/24 | Leaf-3   | Eth2        | 172.16.4.1/24 |

loopback table

| Device  | Loopback ip |
| ------- | ----------- |
| Spine-1 | 192.168.1.1 |
| Spine-2 | 192.168.1.2 |
| Leaf-1  | 192.168.2.1 |
| Leaf-2  | 192.168.2.2 |
| Leaf-3  | 192.168.2.2 |

#### Примечания

{% hint style="info" %}
### так как в явном виде не указано что делать с адресацией на хостовых машинах, а в качестве leaf и spine выступают роутеры, до настройки overlay адреса на хостах распределены из разных подсетей.
{% endhint %}

Когда поднимем overlay засунем все в одну подсеть чтобы эмулировать l2.

#### Запуск лабараторной в среде netlab

Так как у самурая только путь, делаем все в netlab-tools.

[Конфиг-файл](topology.yml) или под катом

<details>

<summary>topology.yml</summary>

```yml
---
provider: clab

nodes:
s1:
device: eos
id: 1
loopback:
  ipv4: 192.168.1.1/32
s2:
device: eos
id: 2
loopback:
  ipv4: 192.168.1.2/32
l1:
device: frr
id: 3
loopback:
  ipv4: 192.168.2.1/32
l2:
device: frr
id: 4
loopback:
  ipv4: 192.168.2.2/32
l3:
device: frr
id: 5
loopback:
  ipv4: 192.168.2.3/32
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
      ipv4: 10.1.1.1
    - node: l1
      ifname: eth8
      ipv4: 10.1.1.2
  prefix:
    ipv4: 10.1.1.0/30
- interfaces:
    - node: s1
      ifname: eth2
      ipv4: 10.1.1.5
    - node: l2
      ifname: eth8
      ipv4: 10.1.1.6
  prefix:
    ipv4: 10.1.1.4/30
- interfaces:
    - node: s1
      ifname: eth3
      ipv4: 10.1.1.9
    - node: l3
      ifname: eth8
      ipv4: 10.1.1.10
  prefix:
    ipv4: 10.1.1.8/30
#spine2-leaf1,2,3
- interfaces:
    - node: s2
      ifname: eth1
      ipv4: 10.1.2.1
    - node: l1
      ifname: eth9
      ipv4: 10.1.2.2
  prefix:
    ipv4: 10.1.2.0/30
- interfaces:
    - node: s2
      ifname: eth2
      ipv4: 10.1.2.5
    - node: l2
      ifname: eth9
      ipv4: 10.1.2.6
  prefix:
    ipv4: 10.1.2.4/30
- interfaces:
    - node: s2
      ifname: eth3
      ipv4: 10.1.2.9
    - node: l3
      ifname: eth9
      ipv4: 10.1.2.10
  prefix:
    ipv4: 10.1.2.8/30
#host1
- interfaces:
    - node: h1
      ifname: eth1
      ipv4: 172.16.1.2
    - node: l1
      ifname: eth1
      ipv4: 172.16.1.1
  prefix:
    ipv4: 172.16.1.0/24
#host2
- interfaces:
    - node: h2
      ifname: eth1
      ipv4: 172.16.2.2
    - node: l2
      ifname: eth1
      ipv4: 172.16.2.1
  prefix:
    ipv4: 172.16.2.0/24
#host3
- interfaces:
    - node: h3
      ifname: eth1
      ipv4: 172.16.3.2
    - node: l3
      ifname: eth1
      ipv4: 172.16.3.1
  prefix:
    ipv4: 172.16.3.0/24
#host4
- interfaces:
    - node: h4
      ifname: eth1
      ipv4: 172.16.4.3
    - node: l3
      ifname: eth2
      ipv4: 172.16.4.1
  prefix:
    ipv4: 172.16.4.0/24
```

</details>

### Проверка работы

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

<summary>spine-2</summary>

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

Пинговать хосты или leaf между собой лишено смысла, т.к. нет маршрутов на устройствах.
