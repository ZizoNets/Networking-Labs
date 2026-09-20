# Design

## 1. Topology

Star topology: the Layer 3 switch (`L3-SW`) acts as the core switch and the default gateway for every department VLAN. Each department is connected to a dedicated Layer 2 access switch. `R1` is the only exit point to the external network.

All links between `L3-SW` and the access switches are configured as **access links**, with each link carrying only the VLAN assigned to that department. No trunk links are used in this design.

| From          | To                    | Link type           | VLAN            |
| ------------- | --------------------- | ------------------- | --------------- |
| L3-SW         | SW1 (IT)              | Access              | 20              |
| L3-SW         | SW2 (Sales)           | Access              | 10              |
| L3-SW         | SW3 (Administration)  | Access              | 60              |
| L3-SW         | SW4 (Management)      | Access              | 50              |
| L3-SW         | SW5 (Human Resources) | Access              | 40              |
| L3-SW         | SW6 (Finance)         | Access              | 30              |
| L3-SW         | R1                    | Routed Layer 3 link | —               |
| R1            | INTERNET              | Simulated           | —               |
| PCs / servers | Their access switch   | Access              | Department VLAN |

## 2. VLANs

| VLAN ID | Name            | Department           | Devices                         | Access switch |
| ------- | --------------- | -------------------- | ------------------------------- | ------------- |
| 10      | Sales           | Sales                | 15 PCs                          | SW2           |
| 20      | IT              | IT                   | 7 PCs + ERP_SERVER + FTP_SERVER | SW1           |
| 30      | Finance         | Finance              | 6 PCs                           | SW6           |
| 40      | Human Resources | HR                   | 4 PCs                           | SW5           |
| 50      | Management      | Executive management | 6 PCs                           | SW4           |
| 60      | Administration  | Administration       | 8 PCs                           | SW3           |

> VLAN 50 "Management" refers to the executive management department. It is not a dedicated network management VLAN.

## 3. VLSM Addressing — 172.20.10.0/24

The `172.20.10.0/24` network is subnetted using VLSM according to the number of hosts required by each department.

| VLAN               | Network          | Mask            | Usable range | Broadcast | Usable hosts |  Needed |
| ------------------ | ---------------- | --------------- | ------------ | --------- | -----------: | ------: |
| 10 Sales           | 172.20.10.0/27   | 255.255.255.224 | .1 – .30     | .31       |           30 | 15 + GW |
| 20 IT              | 172.20.10.32/28  | 255.255.255.240 | .33 – .46    | .47       |           14 |  9 + GW |
| 60 Administration  | 172.20.10.48/28  | 255.255.255.240 | .49 – .62    | .63       |           14 |  8 + GW |
| 50 Management      | 172.20.10.64/28  | 255.255.255.240 | .65 – .78    | .79       |           14 |  6 + GW |
| 30 Finance         | 172.20.10.80/28  | 255.255.255.240 | .81 – .94    | .95       |           14 |  6 + GW |
| 40 Human Resources | 172.20.10.96/29  | 255.255.255.248 | .97 – .102   | .103      |            6 |  4 + GW |
| L3-SW ↔ R1         | 172.20.10.104/30 | 255.255.255.252 | .105 – .106  | .107      |            2 |       2 |

**Free space:** `172.20.10.108 – 172.20.10.255`

All end devices use **static IPv4 addressing**. DHCP is not used.

## 4. SVIs and Default Gateways

The Layer 3 switch provides the default gateway for every department through an SVI for each VLAN.

The default gateway is configured using the **last usable IP address of each subnet**. Each host is manually assigned an IP address from its department subnet and uses the corresponding SVI on `L3-SW` as its default gateway.

| Device | SVI     | IP / Mask        | Default Gateway for Hosts |
| ------ | ------- | ---------------- | ------------------------- |
| L3-SW  | VLAN 10 | 172.20.10.30/27  | 172.20.10.30              |
| L3-SW  | VLAN 20 | 172.20.10.46/28  | 172.20.10.46              |
| L3-SW  | VLAN 30 | 172.20.10.94/28  | 172.20.10.94              |
| L3-SW  | VLAN 40 | 172.20.10.102/29 | 172.20.10.102             |
| L3-SW  | VLAN 50 | 172.20.10.78/28  | 172.20.10.78              |
| L3-SW  | VLAN 60 | 172.20.10.62/28  | 172.20.10.62              |

Inter-VLAN routing is performed by `L3-SW` using these SVIs with `ip routing` enabled.

## 5. Access Links

Each Layer 2 access switch is connected directly to the Layer 3 switch through a dedicated access link.

Both ends of each link are configured as access ports in the same VLAN. Therefore, each physical link carries traffic for only one department.

| Link        | VLAN | Allowed traffic |
| ----------- | ---: | --------------- |
| L3-SW ↔ SW1 |   20 | IT              |
| L3-SW ↔ SW2 |   10 | Sales           |
| L3-SW ↔ SW3 |   60 | Administration  |
| L3-SW ↔ SW4 |   50 | Management      |
| L3-SW ↔ SW5 |   40 | Human Resources |
| L3-SW ↔ SW6 |   30 | Finance         |

No trunking, native VLAN, or VLAN pruning is required because no link carries multiple VLANs.

## 6. Routed Link and Static Routes

The connection between `L3-SW` and `R1` is a routed Layer 3 point-to-point link. The interface on `L3-SW` is configured with `no switchport`.

The VLANs are terminated on the SVIs of `L3-SW`. `R1` does not directly handle the individual VLANs; it only communicates with the Layer 3 switch through the routed point-to-point network.

| Device | Interface        | IP / Mask        | Note                |
| ------ | ---------------- | ---------------- | ------------------- |
| L3-SW  | Routed interface | 172.20.10.106/30 | `no switchport`     |
| R1     | Routed interface | 172.20.10.105/30 | Point-to-point link |

### Static Routes

`L3-SW` uses a default static route pointing to `R1` for traffic destined for external networks.

`R1` uses a **summarized static route** pointing to `L3-SW` for the entire internal `172.20.10.0/24` network. This single route covers all department VLAN subnets.

| Device | Destination network | Mask          | Next hop      |
| ------ | ------------------- | ------------- | ------------- |
| L3-SW  | 0.0.0.0             | 0.0.0.0       | 172.20.10.105 |
| R1     | 172.20.10.0         | 255.255.255.0 | 172.20.10.106 |

