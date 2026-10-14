# Networking-Labs

Networking labs that I build and document as I study. Most of them are Cisco, simulated in GNS3, and each one comes with its design, the configuration of every device and the tests that show it works.

## How each lab is organised

Every lab has its own folder with the same layout:

```
Lab-XXX-Name/
├── Design.md
├── configs/
└── verification/
```

`Design.md` is where a lab starts. It has the scenario, the objectives, the topology, the addressing plan and the reasoning behind the design decisions. Next to it, `configs/` holds one file per device, so each one can be read or reproduced on its own. `verification/` has the tests that confirm everything works: connectivity checks, the commands I run with their expected output and, in some labs, troubleshooting scenarios.

## Environment

The labs run on GNS3 with the GNS3 VM, using Cisco IOS images with the corresponding licensing.

Cisco is my main focus, but I'll add other vendors and technologies over time: Fortinet, Aruba, Juniper, MikroTik, Palo Alto Networks and Ubiquiti, and topics beyond switching and routing such as firewalls, wireless and network monitoring.

## Where it's going

The first labs cover the basics: VLANs, switching, routing, subnetting and inter-VLAN routing. After that they get harder, with dynamic routing, redundancy, security features and troubleshooting.

In each lab I try to understand why a technology is used, how the pieces interact and how to fix it when the result isn't what I expected. The repository is my study log and, at the same time, my portfolio.
