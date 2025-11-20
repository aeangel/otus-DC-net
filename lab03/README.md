# lab03-ISIS-CLOS

### Задание Underlay. IS-IS

Цель: Настроить IS-IS для Underlay сети.

### Схема стенда

![stand-plan](../.gitbook/assets/stand-plan.png)

Стенд делаем по принципу - хосты linux, leaf - frr, spine - eos (arista)

### Распределение адресного пространства для Underlay

План составлен с учетом 10.x.y.z, где x - номер DC, y - номер spine, z - по очереди для подключения leaf Адреса для хостов - 172.16.x.z/24, где x - номер leaf, z - по порядку адрес хоста, на leaf ip .1 Адреса loopback 192.168.a.b/32, где a - 1 для spine, 2 - для leaf, b - номер spine, leaf по порядку Адресацию ipv6 делаем по прицнипу из fd00::\[IPv4]

Interconnect ipv4 ipv6

| Device A | Interface A | IPv4 A        | IPv6 A               | Device B | Interface B | IPv4 B        | IPv6 B               |
| -------- | ----------- | ------------- | -------------------- | -------- | ----------- | ------------- | -------------------- |
| Spine-1  | Eth1        | 10.1.1.0/31   | fd00::10:1:1:0/127   | Leaf-1   | Eth8        | 10.1.1.1/31   | fd00::10:1:1:1/127   |
| Spine-1  | Eth2        | 10.1.1.2/31   | fd00::10:1:1:2/127   | Leaf-2   | Eth8        | 10.1.1.3/31   | fd00::10:1:1:3/127   |
| Spine-1  | Eth3        | 10.1.1.4/31   | fd00::10:1:1:4/127   | Leaf-3   | Eth8        | 10.1.1.5/31   | fd00::10:1:1:5/127   |
| Spine-2  | Eth2        | 10.1.2.0/31   | fd00::10:2:1:0/127   | Leaf-1   | Eth9        | 10.1.2.1/31   | fd00::10:2:1:1/127   |
| Spine-2  | Eth2        | 10.1.2.2/31   | fd00::10:2:1:2/127   | Leaf-2   | Eth9        | 10.1.2.3/31   | fd00::10:2:1:3/127   |
| Spine-2  | Eth3        | 10.1.2.4/31   | fd00::10:2:1:4/127   | Leaf-3   | Eth9        | 10.1.2.5/31   | fd00::10:2:1:5/127   |
| Host-1   | Eth1        | 172.16.1.2/24 | fd00::172:16:1:2/116 | Leaf-1   | Eth1        | 172.16.1.1/24 | fd00::172:16:1:1/116 |
| Host-2   | Eth1        | 172.16.2.2/24 | fd00::172:16:2:2/116 | Leaf-2   | Eth1        | 172.16.2.1/24 | fd00::172:16:2:1/116 |
| Host-3   | Eth1        | 172.16.3.2/24 | fd00::172:16:3:2/116 | Leaf-3   | Eth1        | 172.16.3.1/24 | fd00::172:16:3:1/116 |
| Host-4   | Eth1        | 172.16.4.2/24 | fd00::172:16:4:2/116 | Leaf-3   | Eth2        | 172.16.4.1/24 | fd00::172:16:4:1/116 |

loopback

| Device  | Loopback ipv4 | loopback ipv6     |
| ------- | ------------- | ----------------- |
| Spine-1 | 192.168.1.1   | fd00::192:168:1:1 |
| Spine-2 | 192.168.1.2   | fd00::192:168:1:2 |
| Leaf-1  | 192.168.2.1   | fd00::192:168:2:1 |
| Leaf-2  | 192.168.2.2   | fd00::192:168:2:2 |
| Leaf-3  | 192.168.2.3   | fd00::192:168:2:3 |

isis.net устанавливаем руками, где system id трансляция ip loopback интерфейса, area id 1. Таблица net

| Device  | NET                       |
| ------- | ------------------------- |
| Spine-1 | 49.0001.1921.6801.0001.00 |
| Spine-2 | 49.0001.1921.6801.0002.00 |
| Leaf-1  | 49.0001.1921.6802.0001.00 |
| Leaf-2  | 49.0001.1921.6802.0002.00 |
| Leaf-3  | 49.0001.1921.6802.0003.00 |

{% hint style="info" %}
#### Примечание - так как в явном виде не указано что делать с адресацией на хостовых машинах, а в качестве leaf и spine выступают роутеры, до настройки overlay адреса на хостах распределены из разных подсетей.
{% endhint %}

Когда поднимем overlay засунем все в одну подсеть чтобы эмулировать l2. Из маршрутизации интерфейсы к которым подключены хосты убираем.

### Запуск лабараторной в среде netlab

Заметки по использованию netlab:\
Для работы bfd в frr нужно включить его принудительно в образе по умолчанию, т.к. в netlab не заявлено поддержки bfd на оборудовании frr.\
Правим файл /usr/local/lib/python3.10/dist-packages/netsim/templates/provider/clab/frr/daemons.j2 включая демона - bfdd=yes.\
Так же если хотим использовать другие протоколы в frr, без подключения модулей в netlab в этом же файле нужно включить желаемых демонов.\
Заметки по лабе:\
Возникли проблемы с настройкой bfd для is-is. Так как руками его удалось поднять, пришел к выводу что документация netlab не верна в части заявлений об отсутсвии поддержки bfd для isis для frr интсансов. Залез в модуль /usr/local/lib/python3.10/dist-packages/netsim/ansible/templates/isis/frr.macro.j2 отвечающий за сборку frr и добавил его поддержку. netlab это же про никаких настроек ручками, даже если быстрее и проще? Однако уперся в ограничение со строны frr _Note that there will be just one BFD session per interface. In case both IPv4 and IPv6 support are configured then just a IPv6 based session is created._ Смотрим в документацию по bfd для isis eos(Arista) и узнаем что _BFD is not supported for IPv6 ISIS_. Здесь можно было сдаться, но помня о том что отсутсвие поддержки в документации скорее связано с недоделанным модулем, а не ограничениями платформ пошел проверять руками и без удивления обнаружил работающий bfd для isis на ipv6. Пришлось немного дописать модуль /usr/local/lib/python3.10/dist-packages/netsim/ansible/templates/isis/eos.macro.j2 чтобы все красиво вставало само. Оставшееся ограничение - необходимость в явном виде указывать протокол по которому bfd построится для eos, то есть т.к. со стороны frr у нас dualstack, то на eos нужно явно говорить что работаем с ipv6.

[конфиг-файл](topology.yml) или под катом

<details>

<summary>topology.yml</summary>

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

### Проверка работы

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

<summary>spine-2</summary>

```txt

s2#show ip ro

Gateway of last resort is not set

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

I L2     fd00::10:1:1:0/127 [115/20]
         via fe80::a8c1:abff:fe8b:1dd7, Ethernet1
I L2     fd00::10:1:1:2/127 [115/20]
         via fe80::a8c1:abff:fe10:3ceb, Ethernet2
I L2     fd00::10:1:1:4/127 [115/20]
         via fe80::a8c1:abff:fee5:14ed, Ethernet3
C        fd00::10:1:2:0/127 [0/0]
         via Ethernet1, directly connected
C        fd00::10:1:2:2/127 [0/0]
         via Ethernet2, directly connected
C        fd00::10:1:2:4/127 [0/0]
         via Ethernet3, directly connected
I L2     fd00::192:168:1:1/128 [115/30]
         via fe80::a8c1:abff:fe8b:1dd7, Ethernet1
         via fe80::a8c1:abff:fe10:3ceb, Ethernet2
         via fe80::a8c1:abff:fee5:14ed, Ethernet3
C        fd00::192:168:1:2/128 [0/0]
         via Loopback0, directly connected
I L2     fd00::192:168:2:1/128 [115/20]
         via fe80::a8c1:abff:fe8b:1dd7, Ethernet1
I L2     fd00::192:168:2:2/128 [115/20]
         via fe80::a8c1:abff:fe10:3ceb, Ethernet2
I L2     fd00::192:168:2:3/128 [115/20]
         via fe80::a8c1:abff:fee5:14ed, Ethernet3

s2#show bfd peers
VRF name: default
-----------------
DstAddr                                MyDisc         YourDisc       Interface/Transport         Type               LastUp       LastDown            LastDiag    State
------------------------------- ---------------- ---------------- ------------------------- ------------ -------------------- -------------- ------------------- -----
fe80::a8c1:abff:fe10:3ceb          2399755991       2604022487           Ethernet2(2171)       normal       08/27/25 15:16             NA       No Diagnostic       Up
fe80::a8c1:abff:fe8b:1dd7           854137237       4290215108           Ethernet1(2169)       normal       08/27/25 15:16             NA       No Diagnostic       Up
fe80::a8c1:abff:fee5:14ed          1230089738         45823714           Ethernet3(2179)       normal       08/27/25 15:16             NA       No Diagnostic       Up

s2#show isis neighbors

Instance  VRF      System Id        Type Interface          SNPA              State Hold time   Circuit Id
netlab    default  l1               L2   Ethernet1          P2P               UP    28          00
netlab    default  l2               L2   Ethernet2          P2P               UP    28          00
netlab    default  l3               L2   Ethernet3          P2P               UP    29          00


s2#show isis network topology

IS-IS Instance: netlab VRF: default
IS-IS IPv4 paths to level-2 routers
  System Id        Metric   IA Metric Next-Hop         Interface                SNPA
  s1               20       0         l1               Ethernet1                P2P
                                      l2               Ethernet2                P2P
                                      l3               Ethernet3                P2P
  l1               10       0         l1               Ethernet1                P2P
  l2               10       0         l2               Ethernet2                P2P
  l3               10       0         l3               Ethernet3                P2P
IS-IS IPv6 paths to level-2 routers
  System Id        Metric   IA Metric Next-Hop         Interface                SNPA
  s1               20       0         l1               Ethernet1                P2P
                                      l2               Ethernet2                P2P
                                      l3               Ethernet3                P2P
  l1               10       0         l1               Ethernet1                P2P
  l2               10       0         l2               Ethernet2                P2P
  l3               10       0         l3               Ethernet3                P2P

```

</details>

#### Проверяем пинг loopback всех устройств с leaf-3.

<details>

<summary>leaf-3 pings</summary>

```txt
l3# ping l1
PING l1 (192.168.2.1): 56 data bytes
64 bytes from 192.168.2.1: seq=0 ttl=63 time=1.610 ms
64 bytes from 192.168.2.1: seq=1 ttl=63 time=0.901 ms
64 bytes from 192.168.2.1: seq=2 ttl=63 time=0.808 ms
^C
--- l1 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.808/1.106/1.610 ms
l3# ping l2
PING l2 (192.168.2.2): 56 data bytes
64 bytes from 192.168.2.2: seq=0 ttl=63 time=1.253 ms
64 bytes from 192.168.2.2: seq=1 ttl=63 time=0.875 ms
64 bytes from 192.168.2.2: seq=2 ttl=63 time=0.850 ms
^C
--- l2 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.850/0.992/1.253 ms
l3# ping s1
PING s1 (192.168.1.1): 56 data bytes
64 bytes from 192.168.1.1: seq=0 ttl=64 time=0.074 ms
64 bytes from 192.168.1.1: seq=1 ttl=64 time=0.098 ms
64 bytes from 192.168.1.1: seq=2 ttl=64 time=0.095 ms
64 bytes from 192.168.1.1: seq=3 ttl=64 time=0.092 ms
64 bytes from 192.168.1.1: seq=4 ttl=64 time=0.115 ms
^C
--- s1 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 0.074/0.094/0.115 ms
l3# ping s2
PING s2 (192.168.1.2): 56 data bytes
64 bytes from 192.168.1.2: seq=0 ttl=64 time=0.103 ms
64 bytes from 192.168.1.2: seq=1 ttl=64 time=0.094 ms
64 bytes from 192.168.1.2: seq=2 ttl=64 time=0.173 ms
64 bytes from 192.168.1.2: seq=3 ttl=64 time=0.106 ms
64 bytes from 192.168.1.2: seq=4 ttl=64 time=0.111 ms
^C
--- s2 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 0.094/0.117/0.173 ms

l3# ping ipv6 l1
PING l1 (fd00::192:168:2:1): 56 data bytes
64 bytes from fd00::192:168:2:1: seq=0 ttl=63 time=1.274 ms
64 bytes from fd00::192:168:2:1: seq=1 ttl=63 time=1.009 ms
64 bytes from fd00::192:168:2:1: seq=2 ttl=63 time=0.636 ms
^C
--- l1 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.636/0.973/1.274 ms
l3# ping ipv6 l2
PING l2 (fd00::192:168:2:2): 56 data bytes
64 bytes from fd00::192:168:2:2: seq=0 ttl=63 time=1.395 ms
64 bytes from fd00::192:168:2:2: seq=1 ttl=63 time=1.001 ms
64 bytes from fd00::192:168:2:2: seq=2 ttl=63 time=1.386 ms
^C
--- l2 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 1.001/1.260/1.395 ms
l3# ping ipv6 s1
PING s1 (fd00::192:168:1:1): 56 data bytes
64 bytes from fd00::192:168:1:1: seq=0 ttl=64 time=0.140 ms
64 bytes from fd00::192:168:1:1: seq=1 ttl=64 time=0.115 ms
64 bytes from fd00::192:168:1:1: seq=2 ttl=64 time=0.096 ms
^C
--- s1 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.096/0.117/0.140 ms
l3# ping ipv6 s2
PING s2 (fd00::192:168:1:2): 56 data bytes
64 bytes from fd00::192:168:1:2: seq=0 ttl=64 time=0.105 ms
64 bytes from fd00::192:168:1:2: seq=1 ttl=64 time=0.098 ms
64 bytes from fd00::192:168:1:2: seq=2 ttl=64 time=0.202 ms
64 bytes from fd00::192:168:1:2: seq=3 ttl=64 time=0.090 ms
^C
--- s2 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 0.090/0.123/0.202 ms

```

</details>

Конфигурационные файлы устройств:

[Spine-1](s1.cfg) [Spine-2](s2.cfg) [Leaf-1](l1.cfg) [Leaf-2](l2.cfg) [Leaf-3](l3.cfg)
