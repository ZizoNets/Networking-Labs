# SW1 (Office) Configuration

`SW1` is the access switch for the Office network. It connects the nine Office PCs through access ports and has three additional connections to the rest of the Layer 2 topology: an EtherChannel towards `BackboneSW` and two trunk links towards `SW2` and `SW3`.

<img width="886" height="164" alt="image" src="https://github.com/user-attachments/assets/21d5037e-d8c2-4f7d-88aa-bafb496d6a87" />

The hostname was set to `SW1` and privileged EXEC mode was protected with `enable secret`. VLAN 10 was also created and named `Office`.

<img width="886" height="529" alt="image" src="https://github.com/user-attachments/assets/c4fee03e-7ce4-40cd-af10-9bfc1986ef98" />

The interface status was checked with `do show interface status` before configuring the switch.

Interfaces `G0/0–3`, `G1/0–3` and `G2/0` were configured as access ports in VLAN 10. These nine ports are connected to the Office workstations.

<img width="886" height="416" alt="image" src="https://github.com/user-attachments/assets/a79821c7-28f4-4537-88b3-68e9324e0452" />

VLAN 20 (`Industrial`) was created locally.

`G3/2` was then configured as a trunk towards another switch. The first attempt failed because the trunk encapsulation was still set to `auto`. The interface was configured to use 802.1Q explicitly with `switchport trunk encapsulation dot1q`, after which trunk mode could be enabled.

VLANs 10 and 20 were allowed on the trunk.

<img width="683" height="269" alt="image" src="https://github.com/user-attachments/assets/e35903b0-fd8e-48d8-9685-aed7d5294b5e" />

The same configuration was applied to `G3/1`.

<img width="849" height="141" alt="image" src="https://github.com/user-attachments/assets/c73fed37-93bc-464d-b6c5-37794d1b855f" />

VLAN 999 was configured as the native VLAN on both trunk interfaces. This VLAN is unused and is not used for normal network traffic.

<img width="886" height="175" alt="image" src="https://github.com/user-attachments/assets/6c68fb94-f1f0-4bf1-b01c-5e5ce9272cc4" />

DTP was disabled on both trunk interfaces with `switchport nonegotiate`.

<img width="620" height="94" alt="image" src="https://github.com/user-attachments/assets/1b71a25d-806f-4f44-856a-b3569886bae9" />

Rapid PVST+ was enabled with:

```text id="t6jvym"
spanning-tree mode rapid-pvst
```

<img width="886" height="280" alt="image" src="https://github.com/user-attachments/assets/18089c01-6740-428d-b6ca-bb757dbcc774" />

PortFast and BPDU Guard were enabled on the nine access ports connected to the PCs.

<img width="799" height="63" alt="image" src="https://github.com/user-attachments/assets/1d2d542c-5017-4375-9b3f-5184617a3eea" />

The global PortFast BPDU filter setting was also enabled:

```text id="f4wz2k"
spanning-tree portfast bpdufilter default
```

This applies BPDU filtering by default to interfaces operating as PortFast edge ports.

<img width="672" height="152" alt="image" src="https://github.com/user-attachments/assets/3e6d8f55-c033-42d1-8b30-07711107ba51" />

Interfaces `G2/1–G2/3` and `G3/0` were bundled into a static EtherChannel using `channel-group 1 mode on`. This creates `Port-channel 1` towards `BackboneSW`.

<img width="641" height="175" alt="image" src="https://github.com/user-attachments/assets/0720448e-137f-4fc1-8f37-a1e1111fcbd3" />

`Port-channel 1` was configured as an access port in VLAN 10, matching the configuration on BackboneSW.

<img width="695" height="113" alt="image" src="https://github.com/user-attachments/assets/dd620fae-e7ac-4626-8054-a6cf220fe3e7" />

Loop Guard was enabled on `Port-channel 1` with:

```text id="v4z2wd"
spanning-tree guard loop
```

Since SW1 is not the root bridge, Loop Guard is used on this uplink to help protect against STP loops caused by unexpected BPDU loss.

<img width="706" height="100" alt="image" src="https://github.com/user-attachments/assets/494dbad9-ddb8-49ab-a264-0f2aed0541bf" />

The EtherChannel load-balancing method was set to `src-dst-ip`, matching the configuration on BackboneSW.

<img width="786" height="108" alt="image" src="https://github.com/user-attachments/assets/c23062c8-2562-4a06-9830-3dc09e0d5bcd" />

VTP was configured in transparent mode so VLAN configuration remains local to the switch.

<img width="886" height="109" alt="image" src="https://github.com/user-attachments/assets/d3a2c691-3794-49e0-ac4f-7dd889250518" />

The configuration was saved after completing the switch setup.
