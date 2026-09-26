# Redundant LAN with RSTP & EtherChannel Lab

A Cisco networking lab about building a switched network that refuses to fall over when a cable gets unplugged.

![Lab 002 Topology](images/topology.png)

This lab picks up where **Lab 001** left off with Ferralia Industrial, S.L. The company is opening a small satellite site — an office plus an adjoining industrial unit — and this time a single point of failure isn't an acceptable design. The scenario provided the site layout, the equipment, an IP block, and a not-so-subtle hint that "something broke last time and a truck sat idle in the yard for twenty minutes." Everything else — the redundant Layer 2 design, the EtherChannel bundling, the STP hardening, the VLSM plan, and the routed hand-offs — was designed and built from scratch.

### Technologies & Concepts

* VLANs & 802.1Q Trunking
* Spanning Tree Protocol — PVST+ and Rapid PVST+ (RSTP)
* STP Toolkit: PortFast, BPDU Guard, BPDU Filter, Root Guard, Loop Guard
* EtherChannel (static "on" mode) & load balancing
* Layer 3 Switching, SVIs & Inter-VLAN Routing
* VLSM IPv4 Addressing
* Static Routing
* VTP Transparent Mode
* Native VLAN hardening

### Lab Objectives

* Build a Layer 2 core with no single point of failure between the distribution and access switches.
* Run Rapid PVST+ across the whole fabric with a deliberately chosen, pinned root bridge.
* Turn redundant parallel links into EtherChannels instead of leaving them to STP to block outright.
* Harden every edge port and every switch-facing port against the usual STP mistakes.
* Centralize Inter-VLAN Routing on a single core switch.
* Design the IPv4 addressing plan with VLSM, including the WAN-style point-to-point links.
* Hand off cleanly to a perimeter router and to an external partner network.
* Configure and verify the whole thing end to end.

### Lab Scenario

The new site is split into two functional areas:

* **Office** — 9 workstations, general staff.
* **Industrial Unit** — 3 workstations on the shop floor, next to the loading docks.

The site also needs two outward-facing connections: one to **R1**, the perimeter router that represents the path out to the wider network, and one to **PARTNER-SW**, a switch that belongs to an external logistics partner sharing the same yard. The partner's network is deliberately kept out of scope — it gets a routed hand-off and nothing more, because nobody needs their VLANs bleeding into someone else's infrastructure.

The interesting part of the brief was the middle of the network. Rather than a simple star, the design links **BackboneSW** (the core), **SW1** (Office), **SW2** (Industrial Unit) and **SW3** (a pure distribution switch with no end devices of its own) so that every one of those four switches has a path to every other one. That's on purpose: it turns the Layer 2 core into a small mesh with built-in loops, which is exactly the kind of topology Spanning Tree exists to tame. Two of the links carrying the heaviest traffic (BackboneSW–SW1 and BackboneSW–SW2) are also bundled into EtherChannels, so the design isn't just "redundant," it's redundant *and* faster than a single cable would allow.

### Main Requirements

* Create a dedicated VLAN for the Office and one for the Industrial Unit.
* Interconnect BackboneSW, SW1, SW2 and SW3 so that no single link or switch failure isolates a VLAN.
* Run Rapid PVST+ on every switch and pin BackboneSW as the root bridge for both VLANs.
* Bundle the redundant links toward the core into EtherChannels using static ("on" mode) channel groups.
* Protect access ports with PortFast and BPDU Guard.
* Protect the topology itself with Root Guard on the root's downlinks and Loop Guard on the non-root switches' uplinks.
* Use a dedicated, unused native VLAN on every trunk — no data ever rides the native VLAN.
* Disable DTP negotiation on every trunk link.
* Centralize Inter-VLAN Routing on BackboneSW using SVIs.
* Subnet the assigned `10.60.0.0/26` block with VLSM, including the two /30 point-to-point links.
* Configure a default route toward R1, and a routed (non-trunked) hand-off to PARTNER-SW.
* Configure static IPv4 addressing on all end devices.
* Verify inter-VLAN connectivity end to end and document the topology, addressing, configuration and verification.

### Environment

**GNS3 · GNS3 VM · Cisco IOS**
