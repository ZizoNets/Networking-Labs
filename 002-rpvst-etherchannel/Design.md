# Redundant LAN with RSTP & EtherChannel

A Cisco networking lab focused on building a redundant switched network using Rapid PVST+, EtherChannel and Layer 3 switching.

<img width="1565" height="1102" alt="lab002" src="https://github.com/user-attachments/assets/8cf24dcd-efd2-424e-9fa2-1174ae18f496" />

## 1. Lab Scenario

This lab continues from **Lab 001**, where we built the network for Ferralia Industrial, S.L. The company is now adding a small site consisting of an office and an industrial unit.

The main goal is to avoid a single cable or switch failure taking down part of the network.

The lab starts from the provided topology, equipment and `10.60.0.0/26` address block. From there, the Layer 2 redundancy, EtherChannels, STP protection, VLSM addressing and Layer 3 connections were designed and configured.

The site contains:

* **Office:** 9 workstations.
* **Industrial Unit:** 3 workstations.
* **R1:** upstream/perimeter router.
* **PARTNER-SW:** external logistics partner network.

The partner network remains separate from the internal VLANs through a routed point-to-point connection.

## 2. Topology

The Layer 2 core consists of `BackboneSW`, `SW1`, `SW2` and `SW3`. The switches are interconnected using multiple paths, intentionally creating redundancy and several possible Layer 2 loops.

Rapid PVST+ manages these paths and provides a loop-free forwarding topology while keeping redundant links available.

`BackboneSW` acts as the Layer 3 core and STP root. `SW3` has no end devices and exists mainly to provide additional paths through the topology.

| From       | To         | Link type             | VLAN(s) | Notes           |
| ---------- | ---------- | --------------------- | ------- | --------------- |
| BackboneSW | SW1        | EtherChannel, 4 links | 10      | `src-dst-ip`    |
| BackboneSW | SW2        | EtherChannel, 2 links | 20      | `src-dst-ip`    |
| BackboneSW | SW3        | 802.1Q trunk          | 10, 20  | Native VLAN 999 |
| SW1        | SW3        | 802.1Q trunk          | 10, 20  | Native VLAN 999 |
| SW1        | SW2        | 802.1Q trunk          | 10, 20  | Native VLAN 999 |
| SW2        | SW3        | 802.1Q trunk          | 10, 20  | Native VLAN 999 |
| BackboneSW | R1         | Routed P2P            | —       | 10.60.0.52/30   |
| BackboneSW | PARTNER-SW | Routed P2P            | —       | 10.60.0.48/30   |

## 3. VLAN Design

Only two production VLANs are used:

| VLAN | Name       | Purpose                      | Access switch |
| ---- | ---------- | ---------------------------- | ------------- |
| 10   | Office     | Office workstations          | SW1           |
| 20   | Industrial | Industrial Unit workstations | SW2           |

The native VLAN on the trunks is `999`. It is unused for normal network traffic.

## 4. VLSM Addressing

The site was assigned the `10.60.0.0/26` network. VLSM was used to divide the address space according to the planned host requirements.

For the lab, the number of configured end devices was reduced to avoid spending unnecessary time configuring every host individually. The original host requirements were still used when designing the subnets, so the addressing plan represents the intended capacity of the site and leaves room for future expansion.

| Segment                 | Network       | Usable range | Broadcast | Hosts |
| ----------------------- | ------------- | ------------ | --------- | ----: |
| VLAN 10 Office          | 10.60.0.0/27  | .1 – .30     | .31       |    30 |
| VLAN 20 Industrial      | 10.60.0.32/28 | .33 – .46    | .47       |    14 |
| BackboneSW ↔ PARTNER-SW | 10.60.0.48/30 | .49 – .50    | .51       |     2 |
| BackboneSW ↔ R1         | 10.60.0.52/30 | .53 – .54    | .55       |     2 |

The remaining `10.60.0.56 – 10.60.0.63` is reserved for future use.

All end devices use static IPv4 addressing.

## 5. Layer 3 Design

`BackboneSW` performs inter-VLAN routing using SVIs:

| SVI     | Address       | Purpose            |
| ------- | ------------- | ------------------ |
| VLAN 10 | 10.60.0.30/27 | Office gateway     |
| VLAN 20 | 10.60.0.46/28 | Industrial gateway |

IP routing is enabled on `BackboneSW`.

The connection towards `R1` is a routed point-to-point link:

```text
BackboneSW G0/0  10.60.0.53/30
        |
        |
R1 G1/0          10.60.0.54/30
```

`BackboneSW` uses a default route towards R1:

```text
0.0.0.0/0 → 10.60.0.54
```

The connection to `PARTNER-SW` is also routed:

```text
BackboneSW G0/3  10.60.0.49/30
        |
PARTNER-SW       10.60.0.50/30
```

This keeps the partner network outside the internal VLAN structure.

## 6. EtherChannel Design

Two EtherChannels are used towards the core.

| Port-channel | Connection       | Physical links | Mode | VLAN |
| ------------ | ---------------- | -------------: | ---- | ---- |
| Po1          | BackboneSW ↔ SW1 |              4 | `on` | 10   |
| Po2          | BackboneSW ↔ SW2 |              2 | `on` | 20   |

Both use static EtherChannel configuration without LACP or PAgP.

The load balancing method is:

```text
src-dst-ip
```

This allows different IP flows to be distributed across the physical members of each bundle.

## 7. Spanning Tree Design

All switches run Rapid PVST+:

```text
spanning-tree mode rapid-pvst
```

`BackboneSW` is configured as the root bridge for VLAN 10 and VLAN 20.

The redundant topology allows Rapid PVST+ to block selected paths while keeping alternative paths available if a link fails.

### STP Protection

Access ports connected to end devices use:

```text
spanning-tree portfast
spanning-tree bpduguard enable
```

Root Guard is used on the appropriate Layer 2 downlinks from `BackboneSW`.

Loop Guard is configured on the relevant non-root uplinks. `SW1` and `SW2` use interface-level Loop Guard, while `SW3` uses:

```text
spanning-tree loopguard default
```

The trunk links use native VLAN `999` and DTP is disabled with:

```text
switchport nonegotiate
```

VTP is configured in transparent mode on the switches.

## 8. Verification

The final configuration was verified through interface, STP, EtherChannel and routing checks.

End devices were configured with static IPv4 addresses and connectivity between VLAN 10 and VLAN 20 was tested successfully.

For example, an Office host successfully pinged `10.60.0.33`, an Industrial Unit host. The successful ICMP replies confirmed that inter-VLAN routing through `BackboneSW` was working correctly.

The lab also verifies the intended redundancy of the Layer 2 topology, with Rapid PVST+ controlling the redundant paths and EtherChannel providing bundled links towards the core.

## 9. Environment

**GNS3 · GNS3 VM · Cisco IOS**
