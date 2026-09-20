# Enterprise VLAN Segmentation & Inter-VLAN Routing Lab

A Cisco networking lab simulating the network infrastructure of a mid-sized industrial company.

The network was designed and implemented from scratch based on a real-world style network engineering scenario. The exercise provided the company requirements, available equipment, IP address block, and technical constraints, while the network architecture, VLAN structure, VLSM addressing, topology, and routing design were developed as part of the lab.

### Technologies & Concepts

* VLANs & Layer 2 Segmentation
* VLSM IPv4 Addressing
* Layer 3 Switching & SVIs
* Inter-VLAN Routing
* Routed Point-to-Point Link
* Static Routing
* VTP
* Network Segmentation & Verification

### Lab Objectives

* Segment the corporate network by department.
* Centralize Inter-VLAN Routing on a Layer 3 switch.
* Provide connectivity to internal servers and the external network.
* Design the IPv4 addressing plan using VLSM.
* Configure and verify the complete network from scratch.

<img width="1719" height="1164" alt="lab1" src="https://github.com/user-attachments/assets/c10dff3d-f7d2-4916-b4bd-29c6c18e5452" />

### Lab Scenario

The exercise was based on **Ferralia Industrial, S.L.**, a Spanish company dedicated to the manufacturing and distribution of industrial components such as bearings, hydraulic fittings, and machined parts.

The central office network was designed for **48 devices** distributed across six departments:

* Sales : 15 users
* IT : 7 users + 2 servers
* Administration : 8 users
* Finance : 6 users
* Human Resources : 4 users
* Management : 6 users

The IT department also hosts two internal servers providing ERP and file-sharing services.

The original network was a flat network without segmentation. The objective was to redesign it using departmental VLANs, centralized Inter-VLAN Routing on a Layer 3 switch, and a dedicated perimeter router for external connectivity.

The exercise provided the `172.20.10.0/24` address block. The network architecture, VLAN allocation, VLSM subnetting, device distribution, and routing design were developed as part of the lab.

### Main Requirements

* Create a dedicated VLAN for each department.
* Use VLSM to divide the `172.20.10.0/24` network according to departmental requirements.
* Use a Layer 3 switch as the default gateway for all VLANs and perform Inter-VLAN Routing through SVIs.
* Connect each department through a dedicated Layer 2 access switch.
* Use static access links between the Layer 3 core and the departmental access switches.
* Connect the Layer 3 switch to the perimeter router through a routed `/30` point-to-point link.
* Configure static routing between the internal network and the perimeter router.
* Configure static IPv4 addressing on all end devices.
* Verify local gateway connectivity and inter-VLAN reachability through ICMP testing.
* Document the topology, VLAN structure, addressing plan, configuration, and verification process.

### Environment

**GNS3 · GNS3 VM · Cisco IOS**
