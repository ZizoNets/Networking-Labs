# Design

<img width="1565" height="1102" alt="lab002" src="https://github.com/user-attachments/assets/bb362bed-21a3-4b17-984a-0145367c7cbf" />

## 1. Topology

The site uses a redundant Layer 2 topology between `BackboneSW`, `SW1`, `SW2` and `SW3`. There are several paths between the switches, so Rapid PVST+ is used to prevent Layer 2 loops while keeping redundant paths available.

The links between `BackboneSW` and `SW1`, and between `BackboneSW` and `SW2`, are bundled into EtherChannels. The remaining inter-switch connections are individual 802.1Q trunks.

The connections to `R1` and `PARTNER-SW` are routed point-to-point links. They do not carry VLANs.

| From       | To         | Link type                           | VLAN(s) | Notes                        |
| ---------- | ---------- | ----------------------------------- | ------- | ---------------------------- |
| BackboneSW | SW1        | EtherChannel, access (Po1, 4 links) | 10      | `src-dst-ip` load balancing  |
| BackboneSW | SW2        | EtherChannel, access (Po2, 2 links) | 20      | `src-dst-ip` load balancing  |
| BackboneSW | SW3        | Trunk (single link)                 | 10, 20  | Native VLAN 999              |
| SW1        | SW3        | Trunk (single link)                 | 10, 20  | Native VLAN 999              |
| SW1        | SW2        | Trunk (single link)                 | 10, 20  | Native VLAN 999              |
| SW2        | SW3        | Trunk (single link)                 | 10, 20  | Native VLAN 999              |
| BackboneSW | R1         | Routed P2P                          | —       | 10.60.0.52/30                |
| BackboneSW | PARTNER-SW | Routed P2P                          | —       | 10.60.0.48/30, out of scope  |
| SW1        | 9× PCs     | Access                              | 10      | Office workstations          |
| SW2        | 3× PCs     | Access                              | 20      | Industrial Unit workstations |

The four switches form several Layer 2 loops. Rapid PVST+ selects the forwarding and blocking paths for each VLAN while keeping the redundant links available in case of a failure.

## 2. VLANs

| VLAN ID | Name       | Area                         | Devices        | Access switch |
| ------- | ---------- | ---------------------------- | -------------- | ------------- |
| 10      | Office     | Main office                  | 9 workstations | SW1           |
| 20      | Industrial | Industrial Unit / shop floor | 3 workstations | SW2           |

## 3. VLSM Addressing

The site was assigned the `10.60.0.0/26` network. The address space is divided using VLSM according to the number of hosts planned for each segment.

For the lab, the number of configured end devices was reduced to avoid spending unnecessary time configuring and testing every host individually. The original host requirements were still taken into account when designing the subnets, so the available address space represents the intended capacity of the site and can be used for future expansion.


| Segment                 | Network       | Mask            | Usable range | Broadcast | Usable hosts | Needed |
| ----------------------- | ------------- | --------------- | ------------ | --------- | ------------ | ------ |
| VLAN 10 Office          | 10.60.0.0/27  | 255.255.255.224 | .1 – .30     | .31       | 30           | 9 + GW |
| VLAN 20 Industrial      | 10.60.0.32/28 | 255.255.255.240 | .33 – .46    | .47       | 14           | 3 + GW |
| BackboneSW ↔ PARTNER-SW | 10.60.0.48/30 | 255.255.255.252 | .49 – .50    | .51       | 2            | 2      |
| BackboneSW ↔ R1         | 10.60.0.52/30 | 255.255.255.252 | .53 – .54    | .55       | 2            | 2      |

The remaining address space is:

`10.60.0.56 – 10.60.0.63`

This leaves 8 addresses available for future use.

All end devices use static IPv4 addressing.

## 4. SVIs and Default Gateways

`BackboneSW` performs the Layer 3 functions for the internal VLANs. IP routing is enabled and an SVI is configured for each VLAN.

The last usable address of each subnet is used as the default gateway.

| Device     | SVI     | IP / Mask     | Default gateway for   |
| ---------- | ------- | ------------- | --------------------- |
| BackboneSW | VLAN 10 | 10.60.0.30/27 | Office hosts          |
| BackboneSW | VLAN 20 | 10.60.0.46/28 | Industrial Unit hosts |

Inter-VLAN routing is handled by `BackboneSW`. `R1` only sees the routed connection from the core and does not participate directly in VLAN 10 or VLAN 20.

## 5. EtherChannels

Two groups of physical links are configured as EtherChannels.

Both use static `on` mode, so there is no LACP or PAgP negotiation. The load balancing method is based on source and destination IP addresses.

| Port-channel                 | Switch A : members    | Switch B : members   | Mode | Port type | VLAN |
| ---------------------------- | --------------------- | -------------------- | ---- | --------- | ---- |
| Po1 (BackboneSW) ↔ Po1 (SW1) | BackboneSW: G1/0–G1/3 | SW1: G2/1–G2/3, G3/0 | on   | Access    | 10   |
| Po2 (BackboneSW) ↔ Po1 (SW2) | BackboneSW: G0/1–G0/2 | SW2: G0/3, G1/1      | on   | Access    | 20   |

The Office connection uses four physical links, while the Industrial Unit connection uses two.

## 6. Trunking and Native VLAN

The individual inter-switch links are configured as manually defined 802.1Q trunks.

Two additional settings are used on the trunk links:

* **Native VLAN 999** is reserved and not used for normal network traffic.
* **`switchport nonegotiate`** disables DTP. Trunking is configured manually on both ends of the link.

VLAN 999 is not used by either of the production VLANs.

## 7. Spanning Tree Design

All switches run Rapid PVST+ using:

`spanning-tree mode rapid-pvst`

`BackboneSW` is configured as the root bridge for both VLANs.

### Root Bridge

BackboneSW is configured as the primary root for VLAN 10 and VLAN 20:

```text
spanning-tree vlan 10 root primary
spanning-tree vlan 20 root primary
```

This makes the root bridge selection intentional rather than relying on the default STP election.

### Root Guard

Root Guard is used on the Layer 2 interfaces where the root bridge should remain authoritative.

It is currently configured on BackboneSW's Office EtherChannel and the trunk towards SW3.

The Industrial Unit EtherChannel does not currently have Root Guard applied and could be added for consistency.

The routed connection to `PARTNER-SW` does not use Root Guard because it is a Layer 3 link and therefore does not participate in STP.

### Loop Guard

SW1 and SW2 use Loop Guard on their EtherChannel uplinks towards BackboneSW:

```text
spanning-tree guard loop
```

SW3 uses the global configuration:

```text
spanning-tree loopguard default
```

This provides Loop Guard protection on eligible point-to-point ports without configuring the command individually on every interface.

### Edge Port Protection

The access ports connected to end devices use PortFast and BPDU Guard.

There are 9 PC connections on SW1 and 3 on SW2.

PortFast allows the ports to move directly towards the forwarding state, while BPDU Guard protects them if a BPDU is received.

Both access switches also use:

```text
spanning-tree portfast bpdufilter default
```

This provides BPDU filtering on PortFast-enabled ports.

## 8. Routed Links and Static Routes

`BackboneSW` has two Layer 3 interfaces for connections outside the internal switching domain.

| Device     | Interface | IP / Mask     | Connects to |
| ---------- | --------- | ------------- | ----------- |
| BackboneSW | G0/0      | 10.60.0.53/30 | R1          |
| R1         | G1/0      | 10.60.0.54/30 | BackboneSW  |
| BackboneSW | G0/3      | 10.60.0.49/30 | PARTNER-SW  |

The connection to R1 is used as the default path for traffic leaving the site.

The default route on BackboneSW is:

| Device     | Destination | Mask    | Next hop   |
| ---------- | ----------- | ------- | ---------- |
| BackboneSW | 0.0.0.0     | 0.0.0.0 | 10.60.0.54 |

`R1` and `PARTNER-SW` are outside the main scope of this lab. R1 represents the upstream network, while PARTNER-SW represents the external logistics partner.

Both connections are Layer 3 hand-offs, keeping the external networks separate from the internal VLANs.
