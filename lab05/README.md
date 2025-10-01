### Задание VxLAN. L2 VNI

Цель:
Настроить Overlay на основе VxLAN EVPN для L2 связанности между клиентами.

Описание/Пошаговая инструкция выполнения домашнего задания:
В этой самостоятельной работе мы ожидаем, что вы самостоятельно:

Настроите BGP peering между Leaf и Spine в AF l2vpn evpn
Настроите связанность между клиентами в первой зоне и убедитесь в её наличии
Зафиксируете в документации - план работы, адресное пространство, схему сети, конфигурацию устройств

### Схема стенда

![stand-plan](stand-plan.png)

Стенд делаем по принципу - хосты linux, leaf - frr, spine - eos (arista)

### Распределение адресного пространства для Underlay

План составлен с учетом 10.x.y.z, где x - номер DC, y - номер spine, z - по очереди для подключения leaf
Адреса для хостов - 172.16.x.z/24, где x - номер leaf, z - по порядку адрес хоста, на leaf ip .1
Адреса loopback 192.168.a.b/32, где a - 1 для spine, 2 - для leaf, b - номер spine, leaf по порядку
Адресацию ipv6 делаем по прицнипу из fd00::[IPv4]

Interconnect ipv4 ipv6

| Device A | Interface A | IPv4 A        | IPv6 A               | Device B | Interface B | IPv4 B        | IPv6 B               |
|----------|-------------|---------------|----------------------|----------|-------------|---------------|----------------------|
| Spine-1  | Eth1        | 10.1.1.0/31    | fd00::10:1:1:0/127    | Leaf-1   | Eth1        | 10.1.1.1/31    | fd00::10:1:1:1/127    |
| Spine-1  | Eth2        | 10.1.1.2/31    | fd00::10:1:1:2/127    | Leaf-2   | Eth1        | 10.1.1.3/31    | fd00::10:1:1:3/127    |
| Spine-1  | Eth3        | 10.1.1.4/31    | fd00::10:1:1:4/127    | Leaf-3   | Eth1        | 10.1.1.5/31    | fd00::10:1:1:5/127    |
| Spine-2  | Eth2        | 10.1.2.0/31    | fd00::10:2:1:0/127    | Leaf-1   | Eth2        | 10.1.2.1/31    | fd00::10:2:1:1/127    |
| Spine-2  | Eth2        | 10.1.2.2/31    | fd00::10:2:1:2/127    | Leaf-2   | Eth2        | 10.1.2.3/31    | fd00::10:2:1:3/127    |
| Spine-2  | Eth3        | 10.1.2.4/31    | fd00::10:2:1:4/127    | Leaf-3   | Eth2        | 10.1.2.5/31    | fd00::10:2:1:5/127    |
| Host-1   | Eth1        | 172.16.1.11/24  | fd00::172:16:1:b/116   | Leaf-1   | Eth3        | access vlan red | access vlan red   |
| Host-2   | Eth1        | 172.16.2.12/24  | fd00::172:16:2:c/116   | Leaf-2   | Eth3        | access vlan blue  | access vlan blue    |
| Host-3   | Eth1        | 172.16.1.13/24  | fd00::172:16:3:e/116   | Leaf-3   | Eth3        | access vlan red | access vlan red   | 
| Host-4   | Eth1        | 172.16.2.14/24  | fd00::172:16:4:d/116   | Leaf-3   | Eth4        | access vlan blue  | access vlan blue   |

loopback

| Device | Loopback ipv4| loopback ipv6|
|-------------|---------------|-----------|
| Spine-1  | 192.168.1.1 | fd00::192:168:1:1 |
| Spine-2  | 192.168.1.2 | fd00::192:168:1:2 |
| Leaf-1   | 192.168.2.1 | fd00::192:168:2:1 |
| Leaf-2   | 192.168.2.2 | fd00::192:168:2:2 |
| Leaf-3   | 192.168.2.3 | fd00::192:168:2:3 |

Собираем топологию на базе ospf+ibgp. Area ospf 0, bgp as 65500

### Запуск лабараторной в среде netlab
Казалось бы все просто, но очень долго мучался с работой access на frr. Оказалось что если назначать подключаемые интерфейсы не по порядку, то происходит mismatching и с точки зрения frr настраиваются не те интерфейсы которые настраиваются в бриджинг со стороны linux alpine внутри которого этот frr крутится. 
Так же меня не очень устроило что по умолчанию создается сессия между spine выступающими route reflector, которая не совсем понятно зачем нужна в случае отсутсвия прямого линка между ними.
Поэтому лезем в код, а именно покурив  /usr/local/lib/python3.10/dist-packages/netsim/modules/bgp.py  узнаем что при создании сессий в случае присутсвия роут рефлектора, строится не full mesh а сессии только от RR, но не обрабатываются ситуация сессий с другими RR, добавив условие проверки, получаем что netlab работать перестает. Немного вспомнив курсы по python понимаю, что накосячил с отступами, все поправил - netlab запустился. Вроде все хорошо, но понял что данное изменение безоговорочно убирает сессию, хотя опрос всех знакомых сетевых инженеров так и не принес ответа зачем может понадобится такая сессия, решил что может кому и понадобится, а значит добавляем в обработчик /usr/local/lib/python3.10/dist-packages/netsim/modules/bgp.yml параметр который поможет управлять этим процессом. По умолчанию выставил False. Пару раз пришлось все перезапускать чтобы проверить корректность работы. 
Измененные файлы прилагаю если вдруг кто то найдет их здесь раньше, чем я доберусь до того чтобы законтрибьютить это в проект netlaba. 
![bgp.py](./bgp.py)
![bgp.yml](./bgp.yml)




![конфиг файл](./topology.yml)
или под катом

<details>
  <summary>topology.yml </summary>

  ```yml

  ---
provider: clab
module: [ vlan,vxlan,ospf,bgp,evpn,bfd ]
plugin: [ bgp.session ]

#bgp
bgp.bfd: True
bgp.as: 65500

tools:
  edgeshark:
  graphite:


nodes:
 s1:
  device: eos
  id: 1
  bgp.rr: True
  loopback:
    ipv4: 192.168.1.1/32
    ipv6: fd00::192:168:1:1/128
 s2:
  device: eos
  id: 2
  bgp.rr: True
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
  id: 11
  device: linux
 h2:
  id: 12
  device: linux
 h3:
  id: 13
  device: linux
 h4:
  id: 14
  device: linux

#vlan
vlans:
  red:
    mode: bridge
    prefix:
      ipv4: 172.16.1.0/24
      ipv6: fd00::172:16:1:0/116
  blue:
    mode: bridge
    prefix:
      ipv4: 172.16.2.0/24
      ipv6: fd00::172:16:2:0/116


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
        ifname: eth1
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
        ifname: eth1
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
        ifname: eth1
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
        ifname: eth2
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
        ifname: eth2
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
        ifname: eth2
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
      - node: l1
        ifname: eth3
        vlan.access: red
#host2
  - interfaces:
      - node: h2
        ifname: eth1
      - node: l2
        ifname: eth3
        vlan.access: blue
#host3
  - interfaces:
      - node: h3
        ifname: eth1
      - node: l3
        ifname: eth3
        vlan.access: red
#host4
  - interfaces:
      - node: h4
        ifname: eth1
      - node: l3
        ifname: eth4
        vlan.access: blue

     ```
</details>

  
