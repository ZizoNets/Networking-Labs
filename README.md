# Networking-Labs

Hands-on networking labs built around Cisco technologies, enterprise scenarios, troubleshooting and day-to-day administration.

I'm a networking student and I use this repository to practise, document and track my progress. Each lab is a complete exercise: the design, the configuration of every device, and the tests that prove it works.

## Repository structure

Every lab lives in its own folder and follows the same layout:

```
Lab-XXX-Name/
├── Design.md        # Main design of the lab
├── configs/         # Individual configuration of each device
└── verification/    # Functionality tests and verification
```

### 1. `Design.md`

The starting point of every lab. It contains:

* Context and scenario.
* Objectives and requirements.
* Network topology.
* Addressing plan (IP, VLANs, subnets).
* Design decisions and the reasoning behind them.

### 2. Device configurations

The full configuration of each device (routers, switches, firewalls, etc.), one file per device, so each one can be reviewed and reproduced on its own.

### 3. Verification

The functionality tests that validate the lab:

* Connectivity tests.
* Verification commands and expected output.
* Troubleshooting scenarios, when the lab includes them.

## Lab environment

* **GNS3** with the **GNS3 VM** for network emulation.
* Cisco IOS images with the corresponding licensing.
* Simulated enterprise topologies.

My main focus is Cisco, but I'll gradually add other technologies and vendors as I progress, such as **Fortinet, Aruba, Juniper, MikroTik, Palo Alto Networks and Ubiquiti**. The aim is to work with different architectures, operating systems and vendor-specific implementations, and to cover areas beyond switching and routing like firewalls, wireless and network monitoring.

## Learning approach

The goal is to understand why each technology is used, how the components interact, and how to troubleshoot when the result isn't the expected one, rather than just copy a working configuration.

Early labs cover the fundamentals: **VLANs, switching, routing, subnetting and inter-VLAN routing**. Later labs add more advanced topics: dynamic routing, redundancy, security features and troubleshooting scenarios.

This repository is both my study record and my technical portfolio.
