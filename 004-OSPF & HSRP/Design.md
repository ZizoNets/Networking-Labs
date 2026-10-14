# First-Hop Redundancy with HSRP & OSPF

A Cisco networking lab focused on gateway redundancy with HSRP and dynamic routing with OSPF, using VLSM addressing and router-on-a-stick.

<img width="1707" height="1184" alt="lab004" src="https://github.com/user-attachments/assets/03fc401d-7f03-4ea9-95d3-ba3b1cdc70ea" />


## 1. Lab Scenario

Hotel Complutum is building its network. The IT team has three requirements:

* Keep the hotel departments in separate VLANs.
* Keep the network working if one of the two hotel routers fails.
* Use dynamic routing towards the upstream router instead of static routes.

The hotel only has a small address block, so the addressing has to be tight.

The lab starts from the `10.40.0.0/25` block. From there, the VLSM addressing, HSRP groups and OSPF design were planned and configured.

Planned departments:

* **Guests (Wi-Fi):** 25 devices.
* **Administration and reception:** 10 devices.
* **Security:** 1 camera NVR.

The devices in the lab are:

* **R1 and R2:** the two hotel routers, both connected to every user VLAN.
* **Backbone:** the upstream router. It simulates the provider and originates the default route.
* **SW1:** access switch for Administration and Security.
* **SW2:** access switch for Guests.
* **SW3:** switch that joins R1, R2 and Backbone on a shared segment.

The number of configured end devices was reduced to avoid spending unnecessary time configuring every host individually. The addressing plan still uses the original host requirements, so it represents the intended capacity of the hotel.

## 2. Topology

Each hotel router has a link to both access switches, so either router can serve every VLAN. R1 and R2 are also connected to each other and to Backbone.

| From                         | To                      | Link type               | Notes                                  |
| ---------------------------- | ----------------------- | ----------------------- | -------------------------------------- |
| SW1 `Gi1/1`                  | R1 `Gi1/0`              | 802.1Q trunk            | VLANs 20, 30                           |
| SW1 `Gi1/2`                  | R2 `Gi5/0`              | 802.1Q trunk            | VLANs 20, 30                           |
| SW2 `Gi2/0`                  | R1 `Gi5/0`              | Access port, VLAN 10    | Guests, no subinterfaces               |
| SW2 `Gi1/3`                  | R2 `Gi1/0`              | Access port, VLAN 10    | Guests, no subinterfaces               |
| R1 `Gi3/0`                   | R2 `Gi3/0`              | Routed point-to-point   | `10.40.0.64/30`                        |
| Backbone `Gi1/0`, R1 `Gi4/0`, R2 `Gi4/0` | SW3         | Shared Layer 2 segment  | `10.40.0.56/29`                        |

End devices:

* **SW1:** `Gi0/0–Gi0/2` for the Administration PCs and `Gi0/3` and `Gi1/0` for the Security PCs.
* **SW2:** `Gi0/0–Gi0/3` and `Gi1/0–Gi1/2` for the seven Guests PCs.

`SW3` has no end devices and was left with its default configuration. It only acts as the switch for the shared segment between R1, R2 and Backbone. The OSPF adjacencies over that segment confirm that it is working.

The WAN cloud in the diagram is only decorative. Nothing is configured towards it.

## 3. VLAN Design

Three VLANs are used for the hotel departments:

| VLAN | Name     | Purpose                      | Access switch |
| ---- | -------- | ---------------------------- | ------------- |
| 10   | Guests   | Guest devices                | SW2           |
| 20   | Admin    | Administration and reception | SW1           |
| 30   | Security | Security devices             | SW1           |

The switches have no management IP addresses and are administered through the console, so no management VLAN was created.

## 4. VLSM Addressing

The subnets were assigned from largest to smallest inside `10.40.0.0/25`, using the tightest mask possible. Every VLAN needs room for its hosts plus three router addresses: the HSRP virtual IP, R1 and R2.

| Segment              | Network        | Mask  | Usable range | Broadcast | Needed | Gateway (VIP) |
| -------------------- | -------------- | ----- | ------------ | --------- | -----: | ------------- |
| VLAN 10 Guests       | 10.40.0.0/27   | /27   | .1 – .30     | .31       |     28 | 10.40.0.28    |
| VLAN 20 Admin        | 10.40.0.32/28  | /28   | .33 – .46    | .47       |     13 | 10.40.0.44    |
| VLAN 30 Security     | 10.40.0.48/29  | /29   | .49 – .54    | .55       |      4 | 10.40.0.53    |
| R1 ↔ R2 ↔ Backbone   | 10.40.0.56/29  | /29   | .57 – .62    | .63       |      3 | —             |
| R1 ↔ R2              | 10.40.0.64/30  | /30   | .65 – .66    | .67       |      2 | —             |

Router addresses:

| Segment          | R1          | R2          | Backbone   |
| ---------------- | ----------- | ----------- | ---------- |
| VLAN 10 Guests   | 10.40.0.29  | 10.40.0.30  | —          |
| VLAN 20 Admin    | 10.40.0.46  | 10.40.0.45  | —          |
| VLAN 30 Security | 10.40.0.54  | 10.40.0.51  | —          |
| Shared segment   | 10.40.0.59  | 10.40.0.58  | 10.40.0.57 |
| R1 ↔ R2          | 10.40.0.66  | 10.40.0.65  | —          |

The remaining `10.40.0.68 – 10.40.0.127` is free. As CIDR blocks:

```text
10.40.0.68/30
10.40.0.72/29
10.40.0.80/28
10.40.0.96/27
```

Each router also has a `/32` loopback that is used as its OSPF router ID:

| Router   | Loopback  |
| -------- | --------- |
| R1       | 1.1.1.1   |
| R2       | 2.2.2.2   |
| Backbone | 3.3.3.3   |

All end devices use static IPv4 addressing, with the VIP of their VLAN as default gateway.

## 5. Layer 3 Design

Inter-VLAN routing is done by the hotel routers:

| VLAN | Interface on R1 | Interface on R2 | Method               |
| ---- | --------------- | --------------- | -------------------- |
| 10   | `Gi5/0`         | `Gi1/0`         | Physical interface   |
| 20   | `Gi1/0.2`       | `Gi5/0.20`      | Router-on-a-stick    |
| 30   | `Gi1/0.3`       | `Gi5/0.30`      | Router-on-a-stick    |

Guests have a dedicated cable from each router to `SW2`, so VLAN 10 does not need subinterfaces. VLANs 20 and 30 share one trunk per router, so each VLAN uses a subinterface with `encapsulation dot1q`.

The subinterface number does not have to match the VLAN ID. R1 uses `.2` and `.3`, while R2 uses `.20` and `.30`. The VLAN is defined by the `encapsulation dot1q` command.

## 6. HSRP Design

HSRP runs in the three user VLANs. The group number matches the VLAN ID, and the virtual IP is the default gateway of each VLAN.

| VLAN | Group | Virtual IP | R1            | R2            |
| ---- | ----- | ---------- | ------------- | ------------- |
| 10   | 10    | 10.40.0.28 | Standby (100) | Active (110)  |
| 20   | 20    | 10.40.0.44 | Active (110)  | Standby (100) |
| 30   | 30    | 10.40.0.53 | Active (110)  | Standby (100) |

The roles are split so both routers forward traffic: R1 is Active for Administration and Security, and R2 is Active for Guests. Preempt is configured on all groups on both routers, so the router with the higher priority takes its role back after a failure. HSRP uses the default version 1.

## 7. OSPF Design

All routers run OSPF process 1 in a single area, Area 0.

* **Router ID:** taken from the loopback on each router, without the `router-id` command.
* **Passive interfaces:** all user-facing interfaces and loopbacks are passive, so no OSPF packets are sent to the PCs. Their networks are still advertised.
* **R1 ↔ R2 link:** `ip ospf network point-to-point` on both ends, so there is no DR/BDR election on it.
* **Shared segment `10.40.0.56/29`:** broadcast network with default priorities, so the highest router ID wins. Backbone is DR, R2 is BDR and R1 is DROther. The result also depends on the order in which the routers came up, because DR election is not preemptive.
* **Cost:** `auto-cost reference-bandwidth 10000` on all three routers, so Gigabit interfaces have cost 10.
* **Default route:** Backbone has a static default route to `Null0` and uses `default-information originate`, so it becomes an ASBR and advertises the default route to R1 and R2. R1 and R2 have no static default route.

The WAN is a mock-up, so the default route points to `Null0`. This keeps the route installed on Backbone so it can be advertised. Traffic to Internet addresses is discarded at Backbone, so pings to them are expected to fail.

R1 and R2 are adjacent over both the direct link and the shared segment, so each router has two equal-cost paths to the other router's loopback.

## 8. Verification

Each device page includes the `show` outputs for that device. The main checks were:

* **Layer 2:** `show vlan brief`, `show interfaces status` and `show cdp neighbors` on both switches, and `show interfaces trunk` on `SW1`. All nine active ports on `SW2` are in VLAN 10.
* **HSRP:** `show standby brief` on R1 and R2 shows R1 Active for VLANs 20 and 30 and R2 Active for VLAN 10, and every group has a Standby router.
* **OSPF:** all adjacencies are `FULL`, router IDs come from the loopbacks, and the reference bandwidth is `10000` on all routers.
* **Routing:** R1 and R2 receive `O*E2 0.0.0.0/0` via `10.40.0.57`. Backbone has two equal-cost paths to each user VLAN, one through R1 and one through R2.
* **Connectivity:** PCs in different VLANs can reach each other, the Guests virtual IP and the loopbacks of R2 and Backbone.
* **Failover:** with R1 powered off, `PC8` (Administration) keeps its connectivity. With R2 powered off, `PC1` (Guests) keeps its connectivity.

See [End devices and tests](End-Devices-and-Tests.md) for the ping results and failover tests.

## 9. Issues Found During Verification

Three problems were found and fixed while verifying the lab:

* **HSRP group 10 had two Active routers.** The `SW2` port connected to R1 (`Gi2/0`) was still in VLAN 1, so R1 and R2 could not see each other's hellos in VLAN 10. It was fixed by moving the port to VLAN 10. See [SW2](SW2.md).
* **R2 was not advertising VLANs 20 and 30.** It was missing the two `network` statements, so Backbone only had a path to those VLANs through R1. After adding them, Backbone has two equal-cost paths. See [R2](R2.md).
* **The default route on Backbone pointed to `1.1.1.1`.** That address was also R1's loopback, so the route resolved through R1 instead of a real WAN. It was replaced by a route to `Null0`. See [Backbone](Backbone.md).

## 10. Notes

* **Native VLAN:** `SW1` `Gi1/1` has native VLAN 99, but `Gi1/2` uses the default native VLAN 1, and R2 has a `Gi5/0.99` native subinterface that R1 does not have. The native VLAN is not consistent on both ends of the trunks. It does not affect this lab because VLANs 20 and 30 are tagged, but it should be cleaned up.
* **Loopback addresses:** `1.1.1.1`, `2.2.2.2` and `3.3.3.3` are public addresses. They were used for readability. A real network would use a private range.
* **No ACLs:** all VLANs can reach each other. In a real hotel, access between Guests and the internal VLANs would be filtered.

## 11. Configuration Pages

* [Backbone](Backbone.md)
* [R1](R1.md)
* [R2](R2.md)
* [SW1](SW1.md)
* [SW2](SW2.md)
* [End devices and tests](End-Devices-and-Tests.md)

## 12. Environment

**GNS3 · GNS3 VM · Cisco IOS**
