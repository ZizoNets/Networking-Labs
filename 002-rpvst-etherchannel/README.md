# Redundant LAN with RSTP & EtherChannel Lab

A Cisco networking lab focused on building a redundant switched network using RSTP, EtherChannel and Layer 3 switching.

<img width="1565" height="1102" alt="lab002" src="https://github.com/user-attachments/assets/8cf24dcd-efd2-424e-9fa2-1174ae18f496" />

This lab continues from **Lab 001**, where we built the network for Ferralia Industrial, S.L. This time, the company is adding a small site with an office and an industrial unit.

The main goal is simple: avoid having a single cable or switch failure take down part of the network.

The project starts from the provided topology, equipment and `10.60.0.0/26` address block. From there, the Layer 2 topology, EtherChannel configuration, STP protection, VLSM addressing and Layer 3 connections were designed and configured.

## Technologies & Concepts

* VLANs and 802.1Q trunking
* Rapid PVST+
* EtherChannel load balancing
* Layer 3 switching and SVIs
* VLSM IPv4 addressing
* Static routing
* VTP

## Lab Objectives

* Build a redundant Layer 2 core between the distribution and access switches.
* Use Rapid PVST+ across the network with BackboneSW as the root bridge.
* Use EtherChannel on the redundant links connected to the core.
* Apply STP protections to both end device ports and switch to switch links.
* Centralize inter VLAN routing on BackboneSW using SVIs.
* Create a VLSM addressing plan from the `10.60.0.0/26` network.
* Configure the point to point links using `/30` networks.
* Connect the site to R1 using a routed link and configure the default route.
* Provide a separate routed connection to the external partner network.
* Configure static addressing on the end devices.
* Verify connectivity and STP behaviour across the complete topology.

## Lab Scenario

The site is divided into two main areas:

* **Office**: 9 workstations used by general staff.
* **Industrial Unit**: 3 workstations located on the shop floor.

There are also two external connections. **R1** represents the connection towards the wider network, while **PARTNER-SW** represents a switch belonging to an external logistics partner.

The partner network is kept separate from the internal VLANs. The connection between both networks is routed, so there is no Layer 2 extension between the two infrastructures.

The main part of the lab is the Layer 2 topology.

**BackboneSW**, **SW1**, **SW2** and **SW3** are interconnected using several redundant links. This creates multiple paths between the switches, which means Spanning Tree is required to prevent Layer 2 loops.

The links between **BackboneSW and SW1** and between **BackboneSW and SW2** are also bundled into EtherChannels. This provides redundancy while allowing the links in each bundle to operate as a single logical connection from the point of view of STP.

SW3 does not have any end devices connected to it. Its main purpose in this lab is to provide additional paths through the Layer 2 topology and make the STP behaviour more interesting to verify.

## Main Requirements

* Create one VLAN for the Office and another for the Industrial Unit.
* Provide redundant connectivity between BackboneSW, SW1, SW2 and SW3.
* Run Rapid PVST+ on all switches.
* Configure BackboneSW as the root bridge for both VLANs.
* Configure the redundant links towards BackboneSW as EtherChannels using static `on` mode.
* Enable PortFast and BPDU Guard on access ports.
* Use Root Guard on the appropriate downlinks from the root switch.
* Use Loop Guard on the relevant non root switch uplinks.
* Use a dedicated unused native VLAN on trunk links.
* Disable DTP negotiation on trunk interfaces.
* Keep the native VLAN unused for normal network traffic.
* Configure BackboneSW as the Layer 3 gateway for the internal VLANs using SVIs.
* Subnet the `10.60.0.0/26` address block using VLSM.
* Use `/30` networks for the point to point links.
* Configure a default route from BackboneSW towards R1.
* Configure a routed connection between BackboneSW and PARTNER-SW.
* Keep the partner network outside the internal VLANs.
* Configure static IPv4 addresses on all end devices.
* Test connectivity between VLANs and towards the external networks.
* Verify STP, EtherChannel and routing behaviour.
* Document the topology, addressing plan, configurations and verification results.

## Environment

**GNS3 · GNS3 VM · Cisco IOS**
