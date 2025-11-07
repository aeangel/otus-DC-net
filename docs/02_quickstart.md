# Quick start

Основная страница проекта [https://netlab.tools/](https://netlab.tools/) [https://github.com/ipspace/netlab](https://github.com/ipspace/netlab)

[_netlab_](https://netlab.tools/) приносит подход infrastructure-as-code для создания виртуальных сетевых лабораторных сред. Можно назвать это Lab-as-code решением. Для создания стенда достаточно его описания в yaml.&#x20;

При использовании netlab автоматизирует следующие этапы запуска сетевой лаборатории:

* Создание конфигурации _containerlab_ для запуска Docker контейнеров
* Создание файла Ansible inventory и файлов конфигруации устройств
* Выделение адресации IPv4/IPv6 для стенда, и настройка параметров работы  протоколов маршрутизации OSPFv2, OSPFv3, EIGRP, IS-IS, RIPv2, RIPng, BGP
* Настройка IPv4, IPv6, DHCP, DHCPv6, VLANs, VRFs, VXLAN, LLDP, BFD, OSPFv2, OSPFv3, EIGRP, IS-IS, BGP, RIPv2, RIPng, VRRP, LACP, LAG, MLAG, STP, anycast gateways, static routes, route maps, prefix lists, AS-path prefix lists, route redistribution, default route origination, MPLS, BGP-LU, L3VPN (VPNv4 + VPNv6), 6PE, EVPN, SR-MPLS,  SRv6 на устройствах стенда
* Создание графов и схем топологий, генерация отчетов о работе устройств и протоколов
* Настройка и управление качеством создаваемых виртуальных линков
* Локальный и удаленных захват трафика

Если вам надоело подолгу создавать стенды через GUI чтобы проверить 1-2 вещи, или просто не хотите долго возиться с подготовкой стендов при изучении конекртных технологий netlab отлично вам подойдет.&#x20;
