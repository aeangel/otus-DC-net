![Схема стенда](stand-plan.png)

| Device A  | Interface A | IP A          | Device B  | Interface B | IP B          |
|-----------|-------------|---------------|-----------|-------------|---------------|
| Spine-1  | Gig0/0      | 10.1.1.1/30    | Leaf-1  | Eth0/1      | 10.1.1.2/30    |
| Spine-1  | Eth0/24     | 10.1.1.5/30    | Leaf-2  | Eth0        | 10.1.1.6/30    |
| Spine-1  | Eth0/23     | 10.1.1.9/30    | Leaf-3  | Eth0        | 10.1.1.10/30   |
| Spine-2  | Eth0/22     | 10.1.2.1/30    | Leaf-1  | Eth0        | 10.1.2.2/30    |
| Spine-2  | Gig0/1      | 10.1.2.5/30    | Leaf-2  | WAN         | 10.1.2.6/30    |
| Spine-2  | LAN         | 10.1.2.9/30    | Leaf-3  | Eth0        | 10.1.2.10/30   |
| Host-1  | Eth0/21     | 172.16.0.1/24   | Leaf-1  | Eth0        | none     |
| Host-2  | Gig0/2      | 172.16.0.2/24   | Leaf-2  | Gig0/0      | none       |
| Host-3  | Eth0/21     | 172.16.0.3/24   | Leaf-3  | Eth0        | none     |
| Host-4  | Gig0/2      | 172.16.0.4/24   | Leaf-3  | Gig0/0      | none       |
