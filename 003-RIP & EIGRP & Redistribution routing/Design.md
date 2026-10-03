# RIP, EIGRP & Mutual Redistribution

A Cisco networking lab focused on connecting a RIPv2 domain and an EIGRP domain through a border router, using router-on-a-stick, VLSM, mutual redistribution and a default route propagated from a simulated ISP.
<img width="2240" height="1454" alt="lab003" src="https://github.com/user-attachments/assets/87163be2-1a0c-41c5-a271-1eeb2f3793b9" />


## 1. Lab Scenario

This lab simulates the network of two coffee shops that share a central roastery.

* **Branch A :** the historic shop. One router and one switch, which separates the main room from the back room. Its router only "speaks" **RIP**.
* **Branch B :** the new shop. It runs **EIGRP** and uses two routers so that it does not depend on a single path during peak hours.
* **Server Room (the roastery):** hosts the order servers. It is the only point of the network that speaks both protocols, so it acts as the border router (ASBR).
* **ISP:** simulated with a single router.

The goal is for Branch A and Branch B to see each other, reach the Server Room and reach the Internet.

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
* Four switches are used : `SW1` in Branch A, `SW2` and `SW3` in Branch B (one per LAN) and `SW4` in the Server Room.
* The Server Room has a real LAN with four servers behind `SW4` instead of a loopback.
* Branch A uses router-on-a-stick. Lab 002 used a Layer 3 switch, so this is the different approach.
* Only a few end devices were configured and tested (`PC1`, `PC8`, `PC21`, `Server1`). The addressing plan still uses the full host requirements, so the subnets represent the intended capacity of each LAN.
* Route summarization was not configured. The addressing is contiguous per site so it can be added later (see section 11).

## 3. Topology

Branch A is a single-router site: `R1` connects to `SW1` through one trunk and to `R4` through a routed link.

Branch B has two routers, `R2` and `R3`. Each one has its own LAN and its own link to `R4`, and they are also linked to each other through a serial link. This creates a triangle `R2 – R3 – R4` with two possible paths between each Branch B router and the Server Room.

`R4` connects the three domains: RIP towards `R1`, EIGRP towards `R2` and `R3`, and a static default route towards the ISP.

| From         | To            | Link type             | Network / VLAN   | Notes                           |
| ------------ | ------------- | --------------------- | ---------------- | ------------------------------- |
| R1 G1/0      | SW1 G1/2      | 802.1Q trunk          | VLAN 10, 20      | Native VLAN 999                 |
| R1 G4/0      | R4 G5/0       | Routed P2P            | 192.168.90.0/30  | RIPv2                           |
| R2 G1/0      | R4 G4/0       | Routed P2P            | 192.168.90.4/30  | EIGRP                           |
| R2 S2/0      | R3 S2/0       | Serial, R2 is DCE     | 192.168.90.8/30  | EIGRP, `clock rate 128000`      |
| R3 G3/0      | R4 G1/0       | Routed P2P            | 192.168.90.12/30 | EIGRP                           |
| R4 S2/0      | R5 S2/0       | Serial, R4 is DCE     | 192.168.90.16/30 | `clock rate 128000`, static default |
| R2 Fa0/0     | SW2 G2/2      | Access                | VLAN 30          | Branch B Main gateway           |
| R3 Fa0/0     | SW3 G0/3      | Access                | VLAN 40          | Branch B Terrace gateway        |
| R4 Fa3/0     | SW4           | Access                | VLAN 50          | Server Room gateway             |

## 4. VLAN Design

Five VLANs are used, each one local to a single switch. There are no trunks between sites, so the VLAN IDs are never shared between switches.

| VLAN | Name            | Purpose                  | Switch |
| ---- | --------------- | ------------------------ | ------ |
| 10   | Branch A Coffee | Branch A main room       | SW1    |
| 20   | Branch A BR     | Branch A back room       | SW1    |
| 30   | Branch B Main   | Branch B main room       | SW2    |
| 40   | Branch B Terrace| Branch B terrace / WiFi  | SW3    |
| 50   | Server Room     | Order servers            | SW4    |

The only trunk of the lab is `SW1 ↔ R1`. It uses VLAN `999` as an unused native VLAN, allows only VLANs 10 and 20 and has DTP disabled with `switchport nonegotiate`.

VTP is configured in transparent mode on the switches where it was applied, so VLAN administration remains local to each switch.

## 5. VLSM Addressing

### LAN block: 10.90.0.0/24

| Segment                 | Required | Network        | Mask            | Usable range | Broadcast | Gateway                     |
| ----------------------- | -------: | -------------- | --------------- | ------------ | --------- | --------------------------- |
| VLAN 30 Branch B Main   |       10 | 10.90.0.0/28   | 255.255.255.240 | .1 – .14     | .15       | 10.90.0.14 (R2 Fa0/0)       |
| VLAN 40 Branch B Terrace|        6 | 10.90.0.16/28  | 255.255.255.240 | .17 – .30    | .31       | 10.90.0.30 (R3 Fa0/0)       |
| VLAN 10 Branch A Coffee |        6 | 10.90.0.32/29  | 255.255.255.248 | .33 – .38    | .39       | 10.90.0.38 (R1 G1/0.10)     |
| VLAN 20 Branch A BR     |        2 | 10.90.0.40/29  | 255.255.255.248 | .41 – .46    | .47       | 10.90.0.46 (R1 G1/0.20)     |
| VLAN 50 Server Room     |        4 | 10.90.0.48/29  | 255.255.255.248 | .49 – .54    | .55       | 10.90.0.54 (R4 Fa3/0)       |

The last usable address of each subnet is used as the default gateway.

The LAN prefixes of each site are contiguous, so each site can be represented by a single prefix:

| Site        | Summary prefix | Contains                              |
| ----------- | -------------- | ------------------------------------- |
| Branch B    | 10.90.0.0/27   | 10.90.0.0/28 + 10.90.0.16/28          |
| Branch A    | 10.90.0.32/28  | 10.90.0.32/29 + 10.90.0.40/29         |
| Server Room | 10.90.0.48/29  | 10.90.0.48/29                         |

The remaining `10.90.0.56 – 10.90.0.255` is reserved for future use.

Branch B Terrace needs only six hosts, but it received a `/28` so that the two Branch B LANs fill the `10.90.0.0/27` exactly.

> **Design note:** `Branch A Coffee` uses a `/29` so that both Branch A LANs fit in a single `/28`. A `/29` has six usable addresses and one of them is the gateway, so five addresses remain for the six PCs of the room. Since only one end device per LAN is configured, this does not affect the lab, but a production design would need a `/28` there.

### Link block: 192.168.90.0/24

Every router-to-router link uses a `/30`.

| Link      | Network          | Side A            | Side B            |
| --------- | ---------------- | ----------------- | ----------------- |
| R1 ↔ R4   | 192.168.90.0/30  | R1 `.1`           | R4 `.2`           |
| R2 ↔ R4   | 192.168.90.4/30  | R2 `.5`           | R4 `.6`           |
| R2 ↔ R3   | 192.168.90.8/30  | R2 `.9`           | R3 `.10`          |
| R3 ↔ R4   | 192.168.90.12/30 | R3 `.13`          | R4 `.14`          |
| R4 ↔ R5   | 192.168.90.16/30 | R4 `.17`          | R5 `.18`          |

The remaining `192.168.90.20 – 192.168.90.255` is available for future links.

All end devices use static IPv4 addressing.

## 6. Router-on-a-Stick (Branch A)

Both Branch A VLANs leave through a single physical interface of `R1`, `G1/0`, which is split into two subinterfaces.

| Subinterface | Encapsulation | Address              | Role                    |
| ------------ | ------------- | -------------------- | ----------------------- |
| G1/0.10      | dot1Q 10      | 10.90.0.38/29        | Coffee gateway          |
| G1/0.20      | dot1Q 20      | 10.90.0.46/29        | BR gateway              |

The physical interface has no IP address. Traffic between VLAN 10 and VLAN 20 goes up to `R1` through the trunk and comes back tagged with the other VLAN.

## 7. Routing Design

### 7.1 RIPv2 domain (Branch A)

`R1` runs RIP version 2 with automatic summarization disabled. It advertises `10.0.0.0` and `192.168.90.0`, and the LAN-facing interface `G1/0` is configured as passive so no updates are sent to the users.

`R1` has no static or default route. Everything it knows, including the default route, is learned from `R4`.

### 7.2 EIGRP domain (Branch B)

`R2`, `R3` and `R4` run **EIGRP AS 1** with automatic summarization disabled. Adjacencies are formed only on router-to-router links:

| Link      | Adjacency |
| --------- | --------- |
| R2 ↔ R4   | G1/0 – G4/0 |
| R3 ↔ R4   | G3/0 – G1/0 |
| R2 ↔ R3   | S2/0 – S2/0 |

The LAN interfaces (`R2 Fa0/0`, `R3 Fa0/0`, `R4 Fa3/0`) are passive, so they do not form neighbor relationships. `R2` and `R3` also have a loopback (`2.2.2.2/32` and `3.3.3.3/32`) used as the EIGRP router ID. The loopbacks are passive and advertised as `/32` routes.

`R1` does not take part in EIGRP, and `R2` and `R3` do not run RIP.

#### Path selection

The two Gigabit paths through `R4` are much better than the 128 kbps serial link between `R2` and `R3`. EIGRP uses the default interface bandwidth of the serial link (1544 kbps), because the `bandwidth` command was not changed.

| Path                         | Metric (as seen in the routing tables) |
| ---------------------------- | -------------------------------------: |
| R2 → R4 → R3 LAN (Gigabit)   |                                  28672 |
| R4 → R3 → serial link (R4 view of 192.168.90.8/30) |                2170112 |

`R2` and `R3` reach each other's LAN through `R4`, never through the serial link. `R4` reaches the serial network `192.168.90.8/30` through two equal-cost paths, one via `R2` and one via `R3`.

The serial link is the backup path. The metrics indicate it should be a feasible successor for the LAN of the opposite router: the distance reported by the neighbor (`28160`) is lower than the feasible distance through `R4` (`28672`), so the feasibility condition is met and EIGRP can switch without a new computation. This can be confirmed with:

```text
show ip eigrp topology all-links
```

### 7.3 Border router (R4) and redistribution

`R4` is the only router that runs both protocols, so redistribution is done at a single point. This avoids feedback loops between the two domains.

| Direction          | Command                                             | Seed metric                                          |
| ------------------ | --------------------------------------------------- | ---------------------------------------------------- |
| RIP → EIGRP        | `redistribute rip metric 10000 100 255 1 1500`      | Bandwidth 10000 kbps, delay 100 (1000 µs), reliability 255, load 1, MTU 1500 |
| EIGRP → RIP        | `redistribute eigrp 1 metric 1`                     | Hop count 1                                          |

Seed metric decisions:

* **RIP → EIGRP:** RIP has no EIGRP-style composite metric, so one must be defined. `10000 kbps` and `1000 µs` represent a modest 10 Mbps path: worse than the Gigabit links of the EIGRP domain but better than the 128 kbps serial link. Reliability `255` and load `1` are the best-case values and, with the default K values (`K1=K3=1`), they do not affect the metric. Redistributed routes appear in `R2` and `R3` as external routes (`D EX`, AD 170) with metric `281856`.
* **EIGRP → RIP:** RIP only uses hop count. All the imported routes enter the RIP domain through `R4`, the only exit of `R1`, so the seed value does not change `R1`'s path selection. A hop count of `1` keeps it simple.

Because `network 192.168.90.0` enables RIP on all four `192.168.90.x` interfaces of `R4`, the `R1 ↔ R4` and `R4 ↔ R5` link networks are also redistributed into EIGRP as external routes. The EIGRP loopbacks (`2.2.2.2/32`, `3.3.3.3/32`) and link networks are redistributed into RIP, so `R1` learns them too.

### 7.4 Default route and Internet access

The ISP is simulated by `R5`, connected to `R4` through a serial link.

`R4` is the only router with a default route:

```text
ip route 0.0.0.0 0.0.0.0 192.168.90.18
```

It is propagated to both domains:

| Domain | Method                                               |
| ------ | ---------------------------------------------------- |
| RIP    | `default-information originate`                      |
| EIGRP  | `redistribute static metric 10000 100 255 1 1500`    |

`R1` learns it as a RIP candidate default (`R*`) and `R2` and `R3` as an external EIGRP default (`D*EX`). No manual default routes exist on the café routers.

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

* `R1`: connected networks on `G1/0.10` and `G1/0.20`, RIP routes for Branch B, the Server Room and all link networks via `192.168.90.2`, and `R* 0.0.0.0/0` as gateway of last resort.
* `R2` and `R3`: EIGRP routes for the Server Room and the opposite Branch B router, external routes (`D EX`) for Branch A, and `D*EX 0.0.0.0/0` towards `R4`.
* `R4`: RIP routes for Branch A via `192.168.90.1`, EIGRP routes for Branch B, connected Server Room LAN and the static default towards `192.168.90.18`.

End-to-end connectivity was then tested with ICMP. The TTL of each reply shows how many routers were crossed:

| Source                    | Destination                  | TTL | Routers crossed |
| ------------------------- | ---------------------------- | --: | --------------- |
| PC1 (Branch A Coffee)     | 10.90.0.41 (Branch A BR)     |  63 | R1              |
| PC1 (Branch A Coffee)     | 10.90.0.49 (Server Room)     |  62 | R1, R4          |
| PC1 (Branch A Coffee)     | 10.90.0.17 (Branch B Terrace)|  61 | R1, R4, R3      |
| PC21 (Branch B Terrace)   | 10.90.0.49 (Server Room)     |  62 | R3, R4          |
| PC21 (Branch B Terrace)   | 10.90.0.41 (Branch A BR)     |  61 | R3, R4, R1      |
| PC21 (Branch B Terrace)   | 10.90.0.33 (Branch A Coffee) |  61 | R3, R4, R1      |
| Server1 (Server Room)     | 10.90.0.17 (Branch B Terrace)|  62 | R4, R3          |
| Server1 (Server Room)     | 10.90.0.41 (Branch A BR)     |  62 | R4, R1          |

These tests confirm router-on-a-stick inside Branch A, mutual redistribution between both cafés and access to the Server Room from both domains.

## 10. Troubleshooting

* **Branch A WAN link down:** `GigabitEthernet4/0` of `R1` was administratively down by default. `no shutdown` brought the link up and RIP converged immediately.
* **PC21 could not reach its gateway:** a duplex mismatch (`%CDP-4-DUPLEX_MISMATCH`) was detected between `Fa0/0` of `R3`, which had fallen back to half duplex after autonegotiation, and `G0/3` of `SW3`, which was full duplex. Forcing full duplex on the router and checking that the switch port was an access port in VLAN 40 restored ARP resolution and end-to-end traffic.
* **Wrong mask on the Terrace gateway:** the routing tables captured during verification show the Terrace network as `10.90.0.28/30`, which means `Fa0/0` of `R3` had the mask `255.255.255.252`. It was corrected to `255.255.255.240`, giving the planned `10.90.0.16/28`. The later successful pings to `10.90.0.17` confirm it.

## 11. Known Limitations

* **No route summarization.** The brief asks each site to advertise a single summary route towards the Server Room. This was not configured, so `R4` receives the individual prefixes. The addressing is ready for it: `10.90.0.32/28` for Branch A and `10.90.0.0/27` for Branch B. In Branch A, `R1` is the only exit, so `ip summary-address rip 10.90.0.32 255.255.255.240` on `G4/0` would be enough. In Branch B each router owns one `/28`, so a `/27` summary must be designed carefully to avoid black holes.
* **ISP without return routes.** `R5` only has its connected link. The default route is verified through the routing tables, but `R5` has no route back to the internal networks, so replies from the ISP are not possible yet.
* **One gateway per LAN.** Branch B redundancy is at the routing level (two paths to the Server Room). Each LAN still depends on a single router as gateway because no FHRP is used.
* **RIP on all R4 link interfaces.** `network 192.168.90.0` enables RIP on the links towards `R2`, `R3` and `R5`, where nobody listens. They could be made passive.
* **Passive interface on subinterfaces.** `R1` has `passive-interface GigabitEthernet1/0`, but the LANs use `G1/0.10` and `G1/0.20`. It should be verified with `show ip protocols` that no RIP updates are sent on them, or the subinterfaces should be added explicitly.

## 12. Environment

**GNS3 · GNS3 VM · Cisco IOS**
