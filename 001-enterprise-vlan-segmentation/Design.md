# Enterprise VLAN Segmentation & Inter-VLAN Routing Lab

A Cisco networking lab focused on designing and implementing a segmented enterprise network using VLANs, VLSM, Layer 3 switching and inter-VLAN routing.

<img width="1719" height="1164" alt="lab1" src="https://github.com/user-attachments/assets/f1b71665-38d8-4970-80d9-5ded356516d8" />

## 1. Lab Scenario

This lab simulates the network infrastructure of **Ferralia Industrial, S.L.**, a Spanish company focused on the manufacturing and distribution of industrial components.

The original network was a flat network. The objective was to redesign it using departmental VLANs, centralized inter-VLAN routing and a dedicated perimeter router.

The network was designed from scratch based on the provided requirements, equipment and `172.20.10.0/24` address block.

The central office consists of six departments:

* **Sales:** 15 users
* **IT:** 7 users and 2 servers
* **Administration:** 8 users
* **Finance:** 6 users
* **Human Resources:** 4 users
* **Management:** 6 users

The IT department also contains an ERP server and an FTP server.

## 2. Topology

The network uses a star topology. `L3-SW` acts as the central Layer 3 switch and default gateway for all VLANs. Each department has its own Layer 2 access switch, connected directly to the core through an access link.

`R1` provides the connection towards the external network.

| From  | To                    | Link type  | VLAN |
| ----- | --------------------- | ---------- | ---: |
| L3-SW | SW1 (IT)              | Access     |   20 |
| L3-SW | SW2 (Sales)           | Access     |   10 |
| L3-SW | SW3 (Administration)  | Access     |   60 |
| L3-SW | SW4 (Management)      | Access     |   50 |
| L3-SW | SW5 (Human Resources) | Access     |   40 |
| L3-SW | SW6 (Finance)         | Access     |   30 |
| L3-SW | R1                    | Routed P2P |    — |
| R1    | INTERNET              | Simulated  |    — |

No trunk links are used between the core and access switches. Each link carries traffic for only one department VLAN.

## 3. VLAN Design

Each department is separated into its own VLAN.

| VLAN | Name            | Department           | Devices           | Access switch |
| ---- | --------------- | -------------------- | ----------------- | ------------- |
| 10   | Sales           | Sales                | 15 PCs            | SW2           |
| 20   | IT              | IT                   | 7 PCs + 2 servers | SW1           |
| 30   | Finance         | Finance              | 6 PCs             | SW6           |
| 40   | Human Resources | HR                   | 4 PCs             | SW5           |
| 50   | Management      | Executive management | 6 PCs             | SW4           |
| 60   | Administration  | Administration       | 8 PCs             | SW3           |

`Management` refers to the executive management department and is not a dedicated network management VLAN.

## 4. VLSM Addressing

The assigned `172.20.10.0/24` network is divided using VLSM according to the host requirements of each department and the routed connection to `R1`.

| VLAN / Segment          | Network          | Mask            | Usable range | Broadcast | Hosts |
| ----------------------- | ---------------- | --------------- | ------------ | --------- | ----: |
| VLAN 10 Sales           | 172.20.10.0/27   | 255.255.255.224 | .1 – .30     | .31       |    30 |
| VLAN 20 IT              | 172.20.10.32/28  | 255.255.255.240 | .33 – .46    | .47       |    14 |
| VLAN 60 Administration  | 172.20.10.48/28  | 255.255.255.240 | .49 – .62    | .63       |    14 |
| VLAN 50 Management      | 172.20.10.64/28  | 255.255.255.240 | .65 – .78    | .79       |    14 |
| VLAN 30 Finance         | 172.20.10.80/28  | 255.255.255.240 | .81 – .94    | .95       |    14 |
| VLAN 40 Human Resources | 172.20.10.96/29  | 255.255.255.248 | .97 – .102   | .103      |     6 |
| L3-SW ↔ R1              | 172.20.10.104/30 | 255.255.255.252 | .105 – .106  | .107      |     2 |

The remaining address space, `172.20.10.108 – 172.20.10.255`, is available for future expansion.

All end devices use static IPv4 addressing. DHCP is not used.

## 5. Layer 3 Switching and SVIs

`L3-SW` performs all internal inter-VLAN routing. IP routing is enabled and an SVI is configured for each VLAN.

The last usable address of each subnet is used as the default gateway.

| VLAN | SVI              | Gateway       |
| ---- | ---------------- | ------------- |
| 10   | 172.20.10.30/27  | 172.20.10.30  |
| 20   | 172.20.10.46/28  | 172.20.10.46  |
| 30   | 172.20.10.94/28  | 172.20.10.94  |
| 40   | 172.20.10.102/29 | 172.20.10.102 |
| 50   | 172.20.10.78/28  | 172.20.10.78  |
| 60   | 172.20.10.62/28  | 172.20.10.62  |

The access switches do not perform routing. Traffic between departments is routed by `L3-SW` through the corresponding SVIs.

## 6. Access Links

Each department switch connects to `L3-SW` through a dedicated access link.

Both ends of each link are configured for the same VLAN, so each connection carries traffic for only one department.

| Link        | VLAN | Department      |
| ----------- | ---: | --------------- |
| L3-SW ↔ SW1 |   20 | IT              |
| L3-SW ↔ SW2 |   10 | Sales           |
| L3-SW ↔ SW3 |   60 | Administration  |
| L3-SW ↔ SW4 |   50 | Management      |
| L3-SW ↔ SW5 |   40 | Human Resources |
| L3-SW ↔ SW6 |   30 | Finance         |

Because these are single VLAN access links, trunking, native VLAN configuration and VLAN pruning are not required.

## 7. Routed Link and Static Routing

The connection between `L3-SW` and `R1` is a routed point-to-point link using a `/30` network.

| Device | Interface        | IP address       | Configuration   |
| ------ | ---------------- | ---------------- | --------------- |
| L3-SW  | Routed interface | 172.20.10.106/30 | `no switchport` |
| R1     | Routed interface | 172.20.10.105/30 | Point-to-point  |

`L3-SW` uses a default route towards `R1` for external traffic:

```text
0.0.0.0/0 → 172.20.10.105
```

`R1` uses a summarized static route towards `L3-SW` for the complete internal address block:

```text
172.20.10.0/24 → 172.20.10.106
```

Using a single `/24` route on `R1` covers all six internal VLAN subnets without requiring a separate route for each department.

## 8. Technologies & Concepts

* VLANs and Layer 2 segmentation
* VLSM IPv4 addressing
* Layer 3 switching and SVIs
* Inter-VLAN routing
* Routed point-to-point links
* Static routing
* VTP
* Network segmentation
* ICMP connectivity testing

## 9. Verification

The completed topology was verified by checking the VLAN configuration, SVI status, IP addressing and routing tables.

Connectivity was tested between hosts in different VLANs to confirm that `L3-SW` was correctly routing traffic between the departmental networks. Connectivity towards the routed connection with `R1` was also verified.

## 10. Environment

**GNS3 · GNS3 VM · Cisco IOS**
