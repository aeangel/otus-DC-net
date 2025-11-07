# Список команд

**netlab up** : Uses [**netlab create**](https://netlab.tools/netlab/create/) to create configuration files, starts the virtual lab, and uses [**netlab initial**](https://netlab.tools/netlab/initial/) to deploy device configurations, including IP addressing, LLDP, OSPF, BGP, IS-IS, EIGRP, VRRP, VLANs, VRFs, MPLS, SR-MPLS, VXLAN, EVPN, and SRv6. [More details](https://netlab.tools/netlab/up/)

**netlab down** : Destroys the virtual lab. [More details](https://netlab.tools/netlab/down/)

**netlab restart** : Restart and/or reconfigure the virtual lab. [More details](https://netlab.tools/netlab/restart/)

**netlab config** : [Applies additional Jinja2 configuration templates](https://netlab.tools/netlab/config/) to network devices.

**netlab collect** : Using Ansible fact gathering or other device-specific Ansible modules, [collects device configurations](https://netlab.tools/netlab/collect/) and saves them in the specified directory (default: **config**).

**netlab connect** : Use SSH or **docker exec** to [connect to a lab device](https://netlab.tools/netlab/connect/) using device names, management network IP addresses (**ansible\_host**), SSH port, and username/passwords specified in lab topology or _netlab_ device defaults.

**netlab exec** : Use SSH or **docker exec** to [execute a command on one or more network devices](https://netlab.tools/netlab/exec/) using device names, management network IP addresses (**ansible\_host**), SSH port, and username/passwords specified in lab topology or _netlab_ device defaults.

**netlab capture** : [Perform packet capture](https://netlab.tools/netlab/capture/) on VM- and container interfaces

**netlab tc** : Disable, enable, display, or modify [link impairment](https://netlab.tools/links/#links-netem) parameters

**netlab report** : Creates a report from the transformed lab topology data. [More details](https://netlab.tools/netlab/report/)

**netlab graph** : Creates a lab topology graph description in Graphviz or D2 format. [More details](https://netlab.tools/netlab/graph/)

**netlab show** : Display system settings in tabular, text, or YAML format. [More details](https://netlab.tools/netlab/show/)

**netlab defaults** : Display and manage system defaults. [More details](https://netlab.tools/netlab/defaults/)

**netlab usage** : Display and manage usage statistics. [More details](https://netlab.tools/netlab/usage/)
