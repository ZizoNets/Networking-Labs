# RIP, EIGRP & Mutual Redistribution

A Cisco lab focused on connecting a RIPv2 domain and an EIGRP domain through a border router, using router-on-a-stick, VLSM, mutual redistribution and a default route propagated from a simulated ISP.

<img width="2240" height="1454" alt="lab003" src="https://github.com/user-attachments/assets/87163be2-1a0c-41c5-a271-1eeb2f3793b9" />

## 1. Lab Scenario

This lab simulates the network of two coffee shops that share a central roastery.

* **Branch A:** the historic shop. One router and one switch, which separates the main room from the back room. Its router only "speaks" **RIP**.
* **Branch B:** the new shop. It runs **EIGRP** and uses two routers so that it does not depend on a single path towards the Server Room.
* **Server Room (the roastery):** hosts the order servers. It is the only point of the network that speaks both protocols, so it acts as the border router (ASBR).
* **ISP:** simulated with a single router.

The goal is for Branch A and Branch B to reach each other, reach the Server Room and have a default route towards the simulated ISP.

The network was designed from the provided requirements, equipment and two address blocks: `10.90.0.0/24` for the LANs and `192.168.90.0/24` for the router-to-router links.

| Site        | LAN                | Hosts required |
| ----------- | ------------------ | -------------: |
| Branch A    | Coffee (main room) |              6 |
| Branch A    | BR (back room)     |              2 |
| Branch B    | Main               |             10 |
| Branch B    | Terrace / WiFi     |              6 |
| Server Room | Servers            |              4 |

## 2. Scope and Changes to the Brief

* Five routers (`R1`–`R5`), IPv4 only, no STP and no EtherChannel.
* Four switches are used: `SW1` in Branch A, `SW2` and `SW3` in Branch B (one per LAN), and `SW4` in the Server Room.
* The Server Room has a real LAN with four servers behind `SW4` instead of a loopback.
* Branch A uses router-on-a-stick. Lab 002 used a Layer 3 switch, so this is the different approach.
* Only a few end devices were configured and tested (`PC1`, `PC8`, `PC21`, `Server1`) to save some time. The addressing plan still represents the intended capacity of each LAN.


## 3. Topology

Branch A is a single-router site: `R1` connects to `SW1` through one trunk and to `R4` through a routed link.

Branch B has two routers, `R2` and `R3`. Each one has its own LAN and its own link to `R4`, and they are also linked to each other through a serial link. This creates a triangle `R2 – R3 – R4` with two possible paths between each Branch B router and the Server Room.

`R4` connects the RIP and EIGRP domains and provides the path towards the simulated ISP through a static default route.

| From     | To       | Link type         | Network / VLAN   | Notes                               |
| -------- | -------- | ----------------- | ---------------- | ----------------------------------- |
| R1 G1/0  | SW1 G1/2 | 802.1Q trunk      | VLAN 10, 20      | Native VLAN 999                     |
| R1 G4/0  | R4 G5/0  | Routed P2P        | 192.168.90.0/30  | RIPv2                               |
| R2 G1/0  | R4 G4/0  | Routed P2P        | 192.168.90.4/30  | EIGRP                               |
| R2 S2/0  | R3 S2/0  | Serial, R2 is DCE | 192.168.90.8/30  | EIGRP, `clock rate 128000`          |
| R3 G3/0  | R4 G1/0  | Routed P2P        | 192.168.90.12/30 | EIGRP                               |
| R4 S2/0  | R5 S2/0  | Serial, R4 is DCE | 192.168.90.16/30 | `clock rate 128000`, static default |
| R2 Fa0/0 | SW2 G2/2 | Access            | VLAN 30          | Branch B Main gateway               |
| R3 G1/0  | SW3 G0/3 | Access            | VLAN 40          | Branch B Terrace gateway            |
| R4 Fa3/0 | SW4      | Access            | VLAN 50          | Server Room gateway                 |

## 4. VLAN Design

Five VLANs are used, each one local to a single switch. There are no trunks between sites, so the VLAN IDs are never shared between switches.

| VLAN | Name             | Purpose                 | Switch |
| ---- | ---------------- | ----------------------- | ------ |
| 10   | Branch A Coffee  | Branch A main room      | SW1    |
| 20   | Branch A BR      | Branch A back room      | SW1    |
| 30   | Branch B Main    | Branch B main room      | SW2    |
| 40   | Branch B Terrace | Branch B terrace / WiFi | SW3    |
| 50   | Server Room      | Order servers           | SW4    |

The only trunk of the lab is `SW1 ↔ R1`. It uses VLAN `999` as an unused native VLAN, allows only VLANs 10 and 20, and has DTP disabled with `switchport nonegotiate`.

VTP is configured in transparent mode on all four switches, so VLAN administration remains local to each switch.

## 5. VLSM Addressing

### LAN block: 10.90.0.0/24

| Segment                  | Required | Network       | Mask            | Usable range | Broadcast | Gateway                 |
| ------------------------ | -------: | ------------- | --------------- | ------------ | --------- | ----------------------- |
| VLAN 30 Branch B Main    |       10 | 10.90.0.0/28  | 255.255.255.240 | .1 – .14     | .15       | 10.90.0.14 (R2 Fa0/0)   |
| VLAN 40 Branch B Terrace |        6 | 10.90.0.16/28 | 255.255.255.240 | .17 – .30    | .31       | 10.90.0.30 (R3 G1/0)    |
| VLAN 10 Branch A Coffee  |        6 | 10.90.0.32/29 | 255.255.255.248 | .33 – .38    | .39       | 10.90.0.38 (R1 G1/0.10) |
| VLAN 20 Branch A BR      |        2 | 10.90.0.40/29 | 255.255.255.248 | .41 – .46    | .47       | 10.90.0.46 (R1 G1/0.20) |
| VLAN 50 Server Room      |        4 | 10.90.0.48/29 | 255.255.255.248 | .49 – .54    | .55       | 10.90.0.54 (R4 Fa3/0)   |

The last usable address of each subnet is used as the default gateway.

The LAN prefixes of each site are contiguous, so each site can be represented by a single prefix:

| Site        | Summary prefix | Contains                      |
| ----------- | -------------- | ----------------------------- |
| Branch B    | 10.90.0.0/27   | 10.90.0.0/28 + 10.90.0.16/28  |
| Branch A    | 10.90.0.32/28  | 10.90.0.32/29 + 10.90.0.40/29 |
| Server Room | 10.90.0.48/29  | 10.90.0.48/29                 |

The remaining `10.90.0.56 – 10.90.0.255` is reserved for future use.

Branch B Terrace needs only six hosts, but it received a `/28` so that the two Branch B LANs fill the `10.90.0.0/27` block exactly.

> **Note:** `Branch A Coffee` needs six hosts plus the gateway, which requires seven usable addresses and therefore a `/28`. This was an oversight in the VLSM planning: the LAN received a `/29`, which leaves five usable addresses for end devices once the gateway is assigned. The addressing was not corrected in this lab. Only one end device (`PC1`) is configured on this LAN, so the implemented tests are not affected.

### Link block: 192.168.90.0/24

Every router-to-router link uses a `/30`.

| Link    | Network          | Side A   | Side B   |
| ------- | ---------------- | -------- | -------- |
| R1 ↔ R4 | 192.168.90.0/30  | R1 `.1`  | R4 `.2`  |
| R2 ↔ R4 | 192.168.90.4/30  | R2 `.5`  | R4 `.6`  |
| R2 ↔ R3 | 192.168.90.8/30  | R2 `.9`  | R3 `.10` |
| R3 ↔ R4 | 192.168.90.12/30 | R3 `.13` | R4 `.14` |
| R4 ↔ R5 | 192.168.90.16/30 | R4 `.17` | R5 `.18` |

The remaining `192.168.90.20 – 192.168.90.255` is available for future links.

All end devices use static IPv4 addressing.

## 6. Router-on-a-Stick (Branch A)

Both Branch A VLANs leave through a single physical interface of `R1`, `G1/0`, which is split into two subinterfaces.

| Subinterface | Encapsulation | Address       | Role           |
| ------------ | ------------- | ------------- | -------------- |
| G1/0.10      | dot1Q 10      | 10.90.0.38/29 | Coffee gateway |
| G1/0.20      | dot1Q 20      | 10.90.0.46/29 | BR gateway     |

The physical interface has no IP address. Traffic between VLAN 10 and VLAN 20 goes through `R1` over the 802.1Q trunk.

## 7. Routing Design

### 7.1 RIPv2 domain (Branch A)

`R1` runs RIPv2 with automatic summarization disabled. RIP is enabled on the interfaces belonging to the `10.0.0.0` and `192.168.90.0` address spaces.

`R1` has no static or manually configured default route. Its remote routes, including the default route, are learned from `R4` through RIP.

The LAN subinterfaces `G1/0.10` and `G1/0.20` are configured as passive, so RIP updates are not sent to end devices.

### 7.2 EIGRP domain (Branch B)

`R2`, `R3` and `R4` run **EIGRP AS 1** with automatic summarization disabled. Adjacencies are formed only on router-to-router links:

| Link    | Adjacency   |
| ------- | ----------- |
| R2 ↔ R4 | G1/0 – G4/0 |
| R3 ↔ R4 | G3/0 – G1/0 |
| R2 ↔ R3 | S2/0 – S2/0 |

The LAN interfaces (`R2 Fa0/0`, `R3 G1/0`, `R4 Fa3/0`) are passive, so they do not form EIGRP neighbor relationships.

`R2` and `R3` also have loopbacks (`2.2.2.2/32` and `3.3.3.3/32`) used as their EIGRP router IDs. The loopbacks are passive and advertised as `/32` routes.

`R1` does not take part in EIGRP, and `R2` and `R3` do not run RIP.

#### Path selection

The serial interfaces were configured with a `clock rate` of `128000` on the DCE side. However, the `bandwidth` command was not modified, so EIGRP uses the default interface bandwidth of `1544 kbps` when calculating the metric. The `clock rate` controls serial clocking but does not change the bandwidth value used by EIGRP.

In the routing tables, `R2` and `R3` reach each other's LAN through `R4` and not through the R2–R3 serial link.

The serial link provides an alternative physical path between `R2` and `R3`. Whether it is installed as a feasible successor for a particular destination depends on the EIGRP topology and the feasibility condition.

This can be examined with:

```text
show ip eigrp topology all-links
```

### 7.3 Border router (R4) and redistribution

`R4` is the only router that runs both protocols, so redistribution is performed at a single point. This avoids having multiple redistribution points between the two routing domains.

| Direction   | Command                                        | Seed metric                                                                  |
| ----------- | ---------------------------------------------- | ---------------------------------------------------------------------------- |
| RIP → EIGRP | `redistribute rip metric 10000 100 255 1 1500` | Bandwidth 10000 kbps, delay 100 (1000 µs), reliability 255, load 1, MTU 1500 |
| EIGRP → RIP | `redistribute eigrp 1 metric 1`                | Hop count 1                                                                  |

Seed metric decisions:

* **RIP → EIGRP:** RIP does not provide an EIGRP composite metric, so a seed metric must be defined. `10000 kbps` and `1000 µs` provide a fixed metric for the redistributed routes. Reliability `255` and load `1` are the best-case values and, with the default K values (`K1=K3=1`), they do not affect the metric.
* **EIGRP → RIP:** RIP only uses hop count. All imported routes enter the RIP domain through `R4`, which is the only exit from `R1`, so a seed metric of `1` is sufficient.

The command `network 192.168.90.0` initially enabled RIP on the R4 interfaces belonging to the `192.168.90.0/24` address space. This included the links towards `R2`, `R3` and `R5`. The unnecessary interfaces (`G4/0`, `G1/0` and `S2/0`, towards `R2`, `R3` and `R5`) were subsequently configured as passive after this was identified.

Because RIP is redistributed into EIGRP, RIP-learned or RIP-installed routes can appear in the EIGRP domain as external routes. Similarly, EIGRP routes are redistributed into RIP, allowing `R1` to learn Branch B, Server Room and other redistributed routes.

The EIGRP loopbacks (`2.2.2.2/32` and `3.3.3.3/32`) are also redistributed into RIP.

### 7.4 Default route and Internet access

The ISP is simulated by `R5`, connected to `R4` through a serial link.

`R4` is the only router with a manually configured default route:

```text
ip route 0.0.0.0 0.0.0.0 192.168.90.18
```

The default route is propagated to both routing domains:

| Domain | Method                                            |
| ------ | ------------------------------------------------- |
| RIP    | `default-information originate`                   |
| EIGRP  | `redistribute static metric 10000 100 255 1 1500` |

`R1` learns the default route through RIP, while `R2` and `R3` learn it as an external EIGRP route.

`R5` only has its connected link to `R4`. It is used to simulate the ISP and provide a reachable next hop for the default route. No return routes towards the internal LANs were configured, so full end-to-end Internet connectivity was not part of the test.

## 8. Technologies & Concepts

* VLANs and Layer 2 segmentation
* 802.1Q trunking and router-on-a-stick
* VLSM IPv4 addressing
* RIPv2
* EIGRP, passive interfaces, router ID and feasible successors
* Mutual route redistribution and seed metrics
* Default route propagation
* Serial links and DCE clocking
* ICMP connectivity testing and TTL analysis

## 9. Verification

The routing tables of `R1`, `R2`, `R3` and `R4` were checked after convergence:

* `R1`: connected networks on `G1/0.10` and `G1/0.20`, RIP routes for Branch B, the Server Room and redistributed link networks via `192.168.90.2`, and the RIP-learned default route as the gateway of last resort.
* `R2` and `R3`: EIGRP routes for the Server Room and the opposite Branch B router, external routes (`D EX`) for Branch A, and the external EIGRP default route towards `R4`.
* `R4`: RIP routes for Branch A via `192.168.90.1`, EIGRP routes for Branch B, the connected Server Room LAN, and the static default route towards `192.168.90.18`.

> **Note:** the routing-table captures in [Host Config & Connectivity verification](Host%20Config%20%26%20Conenectivity%20verification.md) were taken before the mask of the Terrace gateway was corrected, so the Terrace network appears there as `10.90.0.28/30` instead of `10.90.0.16/28`. The ICMP tests involving the Terrace LAN were run after the correction.

End-to-end connectivity was then tested with ICMP. The TTL of each reply shows how many routers were crossed:

| Source                  | Destination                   | TTL | Routers crossed |
| ----------------------- | ----------------------------- | --: | --------------- |
| PC1 (Branch A Coffee)   | 10.90.0.41 (Branch A BR)      |  63 | R1              |
| PC1 (Branch A Coffee)   | 10.90.0.49 (Server Room)      |  62 | R1, R4          |
| PC1 (Branch A Coffee)   | 10.90.0.17 (Branch B Terrace) |  61 | R1, R4, R3      |
| PC21 (Branch B Terrace) | 10.90.0.49 (Server Room)      |  62 | R3, R4          |
| PC21 (Branch B Terrace) | 10.90.0.41 (Branch A BR)      |  61 | R3, R4, R1      |
| PC21 (Branch B Terrace) | 10.90.0.33 (Branch A Coffee)  |  61 | R3, R4, R1      |
| Server1 (Server Room)   | 10.90.0.17 (Branch B Terrace) |  62 | R4, R3          |
| Server1 (Server Room)   | 10.90.0.41 (Branch A BR)      |  62 | R4, R1          |

These tests confirm router-on-a-stick inside Branch A, mutual redistribution between the RIP and EIGRP domains, and connectivity between both branches and the Server Room.

## 10. Environment

**GNS3 · GNS3 VM · Cisco IOS**
