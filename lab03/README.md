### Задание Underlay. IS-IS

Цель:
Настроить IS-IS для Underlay сети.

Описание/Пошаговая инструкция выполнения домашнего задания:

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
| Leaf-3   | 192.168.2.3 | fd00::192:168:2:3 |

isis.net устанавливаем руками, где system id  трансляция ip loopback интерфейса, area id 1.
Таблица net 
| Device  | NET                         |
|---------|-----------------------------|
| Spine-1 | 49.0001.1921.6801.0001.00   |
| Spine-2 | 49.0001.1921.6801.0002.00   |
| Leaf-1  | 49.0001.1921.6802.0001.00   |
| Leaf-2  | 49.0001.1921.6802.0002.00   |
| Leaf-3  | 49.0001.1921.6802.0003.00   |


### Примечание - так как в явном виде не указано что делать с адресацией на хостовых машинах, а в качестве leaf и spine выступают роутеры, до настройки overlay адреса на хостах распределены из разных подсетей.
Когда поднимем overlay засунем все в одну подсеть чтобы эмулировать l2. Из маршрутизации интерфейсы к которым подключены хосты убираем.

### Запуск лабараторной в среде netlab
Заметки по использованию netlab:  
Для работы bfd в frr нужно включить его принудительно в образе по умолчанию, т.к. в netlab не заявлено поддержки bfd на оборудовании frr.  
Правим файл /usr/local/lib/python3.10/dist-packages/netsim/templates/provider/clab/frr/daemons.j2 включая демона - bfdd=yes.  
Так же если хотим использовать другие протоколы в frr, без подключения модулей в netlab в этом же файле нужно включить желаемых демонов.  
Заметки по лабе:  
Возникли проблемы с настройкой bfd для is-is. Так как руками его удалось поднять, пришел к выводу что документация netlab  не верна в части заявлений об отсутсвии поддержки bfd для isis для frr интсансов. Залез в модуль /usr/local/lib/python3.10/dist-packages/netsim/ansible/templates/isis/frr.macro.j2 отвечающий за сборку frr и добавил его поддержку. netlab это же про никаких настроек ручками, даже если быстрее и проще?
Однако уперся в ограничение со строны frr _Note that there will be just one BFD session per interface. In case both IPv4 and IPv6 support are configured then just a IPv6 based session is created._ Смотрим в документацию по bfd для isis eos(Arista) и узнаем что _BFD is not supported for IPv6 ISIS_. Здесь можно было сдаться, но помня о том что отсутсвие поддержки в документации скорее связано с недоделанным модулем, а не ограничениями платформ пошел проверять руками и без удивления обнаружил работающий bfd для isis на ipv6. Пришлось немного дописать модуль /usr/local/lib/python3.10/dist-packages/netsim/ansible/templates/isis/eos.macro.j2 чтобы все красиво вставало само. Оставшееся ограничение - необходимость в явном виде указывать протокол по которому bfd построится для eos, то есть т.к. со стороны frr у нас dualstack, то на eos нужно явно говорить что работаем с ipv6. 

![конфиг файл](./topology.yml)
или под катом

<details>
  <summary>topology.yml </summary>

  ```yml
---
provider: clab
module: [ isis]

nodes:
 s1:
  device: eos
  id: 1
  isis:
    instance: netlab
    net: 49.0001.1921.6800.1001.00
  loopback:
    ipv4: 192.168.1.1/32
    ipv6: fd00::192:168:1:1/128
 s2:
  device: eos
  id: 2
  isis:
    instance: netlab
    net: 49.0001.1921.6800.1002.00
  loopback:
    ipv4: 192.168.1.2/32
    ipv6: fd00::192:168:1:2/128
 l1:
  device: frr
  id: 3
  isis:
    instance: netlab
    net: 49.0001.1921.6800.2001.00
  loopback:
    ipv4: 192.168.2.1/32
    ipv6: fd00::192:168:2:1/128
 l2:
  device: frr
  id: 4
  isis:
    instance: netlab
    net: 49.0001.1921.6800.2002.00
  loopback:
    ipv4: 192.168.2.2/32
    ipv6: fd00::192:168:2:2/128
 l3:
  device: frr
  id: 5
  isis:
    instance: netlab
    net: 49.0001.1921.6800.2003.00
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
        isis:
          network_type: point-to-point
          bfd:
            ipv6: True
        ipv4: 10.1.1.0
        ipv6: fd00::10:1:1:0
      - node: l1
        ifname: eth8
        isis:
          network_type: point-to-point
          bfd: True
        ipv4: 10.1.1.1
        ipv6: fd00::10:1:1:1
    prefix:
      ipv4: 10.1.1.0/31
      ipv6: fd00::10:1:1:0/127
  - interfaces:
      - node: s1
        ifname: eth2
        ipv4: 10.1.1.2
        ipv6: fd00::10:1:1:2
        isis:
          network_type: point-to-point
          bfd:
            ipv6: True
      - node: l2
        ifname: eth8
        ipv4: 10.1.1.3
        ipv6: fd00::10:1:1:3
        isis:
          network_type: point-to-point
          bfd: True
    prefix:
      ipv4: 10.1.1.2/31
      ipv6: fd00::10:1:1:2/127
  - interfaces:
      - node: s1
        ifname: eth3
        ipv4: 10.1.1.4
        ipv6: fd00::10:1:1:4
        isis:
          bfd:
            ipv6: True
      - node: l3
        ifname: eth8
        ipv4: 10.1.1.5
        ipv6: fd00::10:1:1:5
        isis:
          bfd: True
    prefix:
      ipv4: 10.1.1.4/31
      ipv6: fd00::10:1:1:4/127
#spine2-leaf1,2,3
  - interfaces:
      - node: s2
        ifname: eth1
        ipv4: 10.1.2.0
        ipv6: fd00::10:1:2:0
        isis:
          network_type: point-to-point
          bfd:
            ipv6: true
      - node: l1
        ifname: eth9
        ipv4: 10.1.2.1
        ipv6: fd00::10:1:2:1
        isis:
          bfd: true
    prefix:
      ipv4: 10.1.2.0/31
      ipv6: fd00::10:1:2:0/127
  - interfaces:
      - node: s2
        ifname: eth2
        ipv4: 10.1.2.2
        ipv6: fd00::10:1:2:2
        isis:
          bfd:
            ipv6: true
      - node: l2
        ifname: eth9
        ipv4: 10.1.2.3
        ipv6: fd00::10:1:2:3
        isis:
          bfd: true
    prefix:
      ipv4: 10.1.2.2/31
      ipv6: fd00::10:1:2:2/127
  - interfaces:
      - node: s2
        ifname: eth3
        ipv4: 10.1.2.4
        ipv6: fd00::10:1:2:4
        isis:
          bfd:
            ipv6: true
      - node: l3
        ifname: eth9
        ipv4: 10.1.2.5
        ipv6: fd00::10:1:2:5
        isis:
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
        isis: false
        ipv4: 172.16.1.1
        ipv6: fd00::172:16:1:1
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
        isis: false
        ipv4: 172.16.2.1
        ipv6: fd00::172:16:2:1
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
        isis: false
        ipv4: 172.16.3.1
        ipv6: fd00::172:16:3:1
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
        isis: false
        ipv4: 172.16.4.1
        ipv6: fd00::172:16:4:1
    prefix:
      ipv4: 172.16.4.0/24
      ipv6: fd00::172:16:4:0/116
```
</details>

## Проверка работы

<details>
  <summary>spine-1</summary>
  
  ```txt  
s1#show ip ro

VRF: default

Gateway of last resort is not set

 C        10.1.1.0/31
           directly connected, Ethernet1
 C        10.1.1.2/31
           directly connected, Ethernet2
 C        10.1.1.4/31
           directly connected, Ethernet3
 I L2     10.1.2.0/31 [115/20]
           via 10.1.1.1, Ethernet1
 I L2     10.1.2.2/31 [115/20]
           via 10.1.1.3, Ethernet2
 I L2     10.1.2.4/31 [115/20]
           via 10.1.1.5, Ethernet3
 C        192.168.1.1/32
           directly connected, Loopback0
 I L2     192.168.1.2/32 [115/30]
           via 10.1.1.1, Ethernet1
           via 10.1.1.3, Ethernet2
           via 10.1.1.5, Ethernet3
 I L2     192.168.2.1/32 [115/20]
           via 10.1.1.1, Ethernet1
 I L2     192.168.2.2/32 [115/20]
           via 10.1.1.3, Ethernet2
 I L2     192.168.2.3/32 [115/20]
           via 10.1.1.5, Ethernet3


s1#show ipv6 ro

VRF: default

 C        fd00::10:1:1:0/127 [0/0]
           via Ethernet1, directly connected
 C        fd00::10:1:1:2/127 [0/0]
           via Ethernet2, directly connected
 C        fd00::10:1:1:4/127 [0/0]
           via Ethernet3, directly connected
 I L2     fd00::10:1:2:0/127 [115/20]
           via fe80::a8c1:abff:fef4:6f56, Ethernet1
 I L2     fd00::10:1:2:2/127 [115/20]
           via fe80::a8c1:abff:fe01:dde1, Ethernet2
 I L2     fd00::10:1:2:4/127 [115/20]
           via fe80::a8c1:abff:fe7b:3d0c, Ethernet3
 C        fd00::192:168:1:1/128 [0/0]
           via Loopback0, directly connected
 I L2     fd00::192:168:1:2/128 [115/30]
           via fe80::a8c1:abff:fef4:6f56, Ethernet1
           via fe80::a8c1:abff:fe01:dde1, Ethernet2
           via fe80::a8c1:abff:fe7b:3d0c, Ethernet3
 I L2     fd00::192:168:2:1/128 [115/20]
           via fe80::a8c1:abff:fef4:6f56, Ethernet1
 I L2     fd00::192:168:2:2/128 [115/20]
           via fe80::a8c1:abff:fe01:dde1, Ethernet2
 I L2     fd00::192:168:2:3/128 [115/20]
           via fe80::a8c1:abff:fe7b:3d0c, Ethernet3

s1#show bfd peers
VRF name: default
------------------
DstAddr                                MyDisc         YourDisc       Interface/Transport         Type               LastUp       LastDown            LastDiag    State
------------------------------- ---------------- ---------------- ------------------------- ------------ -------------------- -------------- ------------------- -----
fe80::a8c1:abff:fe60:94a3          2730906329       2680068865           Ethernet2(2177)       normal       08/27/25 15:43             NA       No Diagnostic       Up
fe80::a8c1:abff:fe88:9959          2530925411       1836350440           Ethernet3(2183)       normal       08/27/25 15:16             NA       No Diagnostic       Up
fe80::a8c1:abff:fe92:99c2          1861245626       1908179683           Ethernet1(2173)       normal       08/27/25 15:16             NA       No Diagnostic       Up

s1#show isis neighbors

Instance  VRF      System Id        Type Interface          SNPA              State Hold time   Circuit Id
netlab    default  l1               L2   Ethernet1          P2P               UP    29          00
netlab    default  l2               L2   Ethernet2          P2P               UP    30          00
netlab    default  l3               L2   Ethernet3          P2P               UP    28          00

s1#show isis network topology

IS-IS Instance: netlab VRF: default
  IS-IS IPv4 paths to level-2 routers
    System Id        Metric   IA Metric Next-Hop         Interface                SNPA
    s2               20       0         l1               Ethernet1                P2P
                                        l2               Ethernet2                P2P
                                        l3               Ethernet3                P2P
    l1               10       0         l1               Ethernet1                P2P
    l2               10       0         l2               Ethernet2                P2P
    l3               10       0         l3               Ethernet3                P2P
  IS-IS IPv6 paths to level-2 routers
    System Id        Metric   IA Metric Next-Hop         Interface                SNPA
    s2               20       0         l1               Ethernet1                P2P
                                        l2               Ethernet2                P2P
                                        l3               Ethernet3                P2P
    l1               10       0         l1               Ethernet1                P2P
    l2               10       0         l2               Ethernet2                P2P
    l3               10       0         l3               Ethernet3                P2P

```
</details>

<details>
  <summary>spine-2 </summary>

  ```txt  

 I L2     10.1.1.0/31 [115/20]
           via 10.1.2.1, Ethernet1
 I L2     10.1.1.2/31 [115/20]
           via 10.1.2.3, Ethernet2
 I L2     10.1.1.4/31 [115/20]
           via 10.1.2.5, Ethernet3
 C        10.1.2.0/31
           directly connected, Ethernet1
 C        10.1.2.2/31
           directly connected, Ethernet2
 C        10.1.2.4/31
           directly connected, Ethernet3
 I L2     192.168.1.1/32 [115/30]
           via 10.1.2.1, Ethernet1
           via 10.1.2.3, Ethernet2
           via 10.1.2.5, Ethernet3
 C        192.168.1.2/32
           directly connected, Loopback0
 I L2     192.168.2.1/32 [115/20]
           via 10.1.2.1, Ethernet1
 I L2     192.168.2.2/32 [115/20]
           via 10.1.2.3, Ethernet2
 I L2     192.168.2.3/32 [115/20]
           via 10.1.2.5, Ethernet3
           
s2#show ipv6 ro

VRF: default
Displaying 17 of 23 IPv6 routing table entries

 O3       fd00::10:1:1:1/128 [110/10]
           via fe80::a8c1:abff:fe45:5487, Ethernet1
 O3       fd00::10:1:1:0/127 [110/20]
           via fe80::a8c1:abff:fe45:5487, Ethernet1
 O3       fd00::10:1:1:3/128 [110/10]
           via fe80::a8c1:abff:feab:149e, Ethernet2
 O3       fd00::10:1:1:2/127 [110/20]
           via fe80::a8c1:abff:feab:149e, Ethernet2
 O3       fd00::10:1:1:5/128 [110/10]
           via fe80::a8c1:abff:fe2a:bc56, Ethernet3
 O3       fd00::10:1:1:4/127 [110/20]
           via fe80::a8c1:abff:fe2a:bc56, Ethernet3
 O3       fd00::10:1:2:1/128 [110/10]
           via fe80::a8c1:abff:fe45:5487, Ethernet1
 C        fd00::10:1:2:0/127 [0/0]
           via Ethernet1, directly connected
 O3       fd00::10:1:2:3/128 [110/10]
           via fe80::a8c1:abff:feab:149e, Ethernet2
 C        fd00::10:1:2:2/127 [0/0]
           via Ethernet2, directly connected
 O3       fd00::10:1:2:5/128 [110/10]
           via fe80::a8c1:abff:fe2a:bc56, Ethernet3
 C        fd00::10:1:2:4/127 [0/0]
           via Ethernet3, directly connected
 O3       fd00::192:168:1:1/128 [110/30]
           via fe80::a8c1:abff:fe45:5487, Ethernet1
           via fe80::a8c1:abff:feab:149e, Ethernet2
           via fe80::a8c1:abff:fe2a:bc56, Ethernet3
 C        fd00::192:168:1:2/128 [0/0]
           via Loopback0, directly connected
 O3       fd00::192:168:2:1/128 [110/10]
           via fe80::a8c1:abff:fe45:5487, Ethernet1
 O3       fd00::192:168:2:2/128 [110/10]
           via fe80::a8c1:abff:feab:149e, Ethernet2
 O3       fd00::192:168:2:3/128 [110/10]
           via fe80::a8c1:abff:fe2a:bc56, Ethernet3

s2#show bfd peers
VRF name: default
-----------------
DstAddr               MyDisc         YourDisc       Interface/Transport         Type               LastUp       LastDown            LastDiag    State
-------------- ---------------- ---------------- ------------------------- ------------ -------------------- -------------- ------------------- -----
10.1.2.1          1038882734       1867128317           Ethernet1(1140)       normal       08/20/25 22:21             NA       No Diagnostic       Up
10.1.2.3           476357661       2044274492           Ethernet2(1144)       normal       08/20/25 22:21             NA       No Diagnostic       Up
10.1.2.5          2107959149       3063445977           Ethernet3(1146)       normal       08/20/25 22:21             NA       No Diagnostic       Up

DstAddr                                MyDisc         YourDisc       Interface/Transport         Type               LastUp       LastDown            LastDiag    State
------------------------------- ---------------- ---------------- ------------------------- ------------ -------------------- -------------- ------------------- -----
fe80::a8c1:abff:fe2a:bc56           755184907       3120229879           Ethernet3(1146)       normal       08/20/25 22:21             NA       No Diagnostic       Up
fe80::a8c1:abff:fe45:5487          2205458108       3120229879           Ethernet1(1140)       normal       08/20/25 22:21             NA       No Diagnostic       Up
fe80::a8c1:abff:feab:149e          1370192998       1612082641           Ethernet2(1144)       normal       08/20/25 22:21             NA       No Diagnostic       Up

```
</details>

### Проверяем пинг loopback всех устройств с leaf-1.

<details>
  <summary>leaf-1 pings </summary>

  ```txt  
l1# ping 192.168.2.2
PING 192.168.2.2 (192.168.2.2): 56 data bytes
64 bytes from 192.168.2.2: seq=0 ttl=63 time=0.813 ms
64 bytes from 192.168.2.2: seq=1 ttl=63 time=0.961 ms
64 bytes from 192.168.2.2: seq=2 ttl=63 time=0.964 ms
64 bytes from 192.168.2.2: seq=3 ttl=63 time=0.932 ms
^C
--- 192.168.2.2 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 0.813/0.917/0.964 ms
l1# ping 192.168.2.3
PING 192.168.2.3 (192.168.2.3): 56 data bytes
64 bytes from 192.168.2.3: seq=0 ttl=63 time=1.186 ms
64 bytes from 192.168.2.3: seq=1 ttl=63 time=1.043 ms
64 bytes from 192.168.2.3: seq=2 ttl=63 time=1.222 ms
64 bytes from 192.168.2.3: seq=3 ttl=63 time=1.048 ms
^C
--- 192.168.2.3 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 1.043/1.124/1.222 ms
l1# ping 192.168.1.1
PING 192.168.1.1 (192.168.1.1): 56 data bytes
64 bytes from 192.168.1.1: seq=0 ttl=64 time=0.145 ms
64 bytes from 192.168.1.1: seq=1 ttl=64 time=0.118 ms
64 bytes from 192.168.1.1: seq=2 ttl=64 time=0.099 ms
64 bytes from 192.168.1.1: seq=3 ttl=64 time=0.098 ms
^C
--- 192.168.1.1 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 0.098/0.115/0.145 ms
l1# ping 192.168.1.2
PING 192.168.1.2 (192.168.1.2): 56 data bytes
64 bytes from 192.168.1.2: seq=0 ttl=64 time=0.102 ms
64 bytes from 192.168.1.2: seq=1 ttl=64 time=0.101 ms
64 bytes from 192.168.1.2: seq=2 ttl=64 time=0.101 ms
64 bytes from 192.168.1.2: seq=3 ttl=64 time=0.112 ms
^C
--- 192.168.1.2 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 0.101/0.104/0.112 ms
l1# ping ipv6 fd00::192:168:2:2
PING fd00::192:168:2:2 (fd00::192:168:2:2): 56 data bytes
64 bytes from fd00::192:168:2:2: seq=0 ttl=63 time=1.543 ms
64 bytes from fd00::192:168:2:2: seq=1 ttl=63 time=0.781 ms
64 bytes from fd00::192:168:2:2: seq=2 ttl=63 time=1.588 ms
^C
--- fd00::192:168:2:2 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.781/1.304/1.588 ms
l1# ping ipv6 fd00::192:168:2:3
PING fd00::192:168:2:3 (fd00::192:168:2:3): 56 data bytes
64 bytes from fd00::192:168:2:3: seq=0 ttl=63 time=1.360 ms
64 bytes from fd00::192:168:2:3: seq=1 ttl=63 time=0.760 ms
64 bytes from fd00::192:168:2:3: seq=2 ttl=63 time=0.913 ms
^C
--- fd00::192:168:2:3 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.760/1.011/1.360 ms
l1# ping ipv6 fd00::192:168:1:1
PING fd00::192:168:1:1 (fd00::192:168:1:1): 56 data bytes
64 bytes from fd00::192:168:1:1: seq=0 ttl=64 time=0.096 ms
64 bytes from fd00::192:168:1:1: seq=1 ttl=64 time=0.149 ms
64 bytes from fd00::192:168:1:1: seq=2 ttl=64 time=0.101 ms
^C
--- fd00::192:168:1:1 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.096/0.115/0.149 ms
l1# ping ipv6 fd00::192:168:1:2
PING fd00::192:168:1:2 (fd00::192:168:1:2): 56 data bytes
64 bytes from fd00::192:168:1:2: seq=0 ttl=64 time=0.111 ms
64 bytes from fd00::192:168:1:2: seq=1 ttl=64 time=0.128 ms
64 bytes from fd00::192:168:1:2: seq=2 ttl=64 time=0.116 ms
64 bytes from fd00::192:168:1:2: seq=3 ttl=64 time=0.097 ms
^C
--- fd00::192:168:1:2 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 0.097/0.113/0.128 ms

```
</details>

Конфигурационные файлы устройств:  
![Leaf-1](./l1.conf)
![Leaf-2](./l2.conf)
![Leaf-3](./l3.conf)
![Spine-1](./s1.conf)
![Spine-2](./s2.conf)

