# SW2 (Industrial Unit) Configuration

`SW2` is the access switch for the Industrial Unit. It connects the three industrial workstations and has redundant connections to the rest of the Layer 2 topology through an EtherChannel towards `BackboneSW` and two trunk links towards `SW1` and `SW3`.

<img width="886" height="86" alt="image" src="https://github.com/user-attachments/assets/3774e788-7fb6-471d-bbe9-c56d9c096718" />

The hostname was set to `SW2` and privileged EXEC mode was protected with `enable secret`.

<img width="841" height="263" alt="image" src="https://github.com/user-attachments/assets/f795064d-81ec-4b2d-8af6-60753ff649f4" />

Interfaces `G0/0–G0/2` were configured as access ports for the three Industrial Unit workstations.

VLAN 20 did not exist on the switch at this point, so IOS created it automatically when it was assigned to the interfaces. It was later configured explicitly as `Industrial`.

PortFast was enabled on the same interfaces because they are connected directly to end devices.

<img width="800" height="47" alt="image" src="https://github.com/user-attachments/assets/6fde4368-4b12-4fa2-9a56-0e09571dab93" />

BPDU Guard was enabled on the three access ports:

```text id="p8n3xw"
spanning-tree bpduguard enable
```

This protects the edge ports from receiving unexpected BPDUs.

<img width="816" height="211" alt="image" src="https://github.com/user-attachments/assets/4245b650-01bc-43bd-9863-5aa9ad11e428" />

Loop Guard was configured on the future EtherChannel towards BackboneSW.

The global PortFast BPDU filter setting was also enabled:

```text id="k6q2fj"
spanning-tree portfast bpdufilter default
```

This matches the configuration used on SW1.

<img width="636" height="150" alt="image" src="https://github.com/user-attachments/assets/dc240152-dbe2-46f3-aafd-b8beab91ad9d" />

Rapid PVST+ was enabled and VTP was configured in transparent mode. VLAN 20 was also explicitly named `Industrial`.

<img width="747" height="111" alt="image" src="https://github.com/user-attachments/assets/0c66d4d0-6205-4448-821f-0ae494b0bde7" />

Interfaces `G0/3` and `G1/1` were bundled into a static EtherChannel using `channel-group 1 mode on`. This creates `Port-channel 1` towards BackboneSW.

<img width="572" height="206" alt="image" src="https://github.com/user-attachments/assets/6a009784-f775-4973-a2cd-55b3a79e81ba" />

`Port-channel 1` was configured as an access port in VLAN 20, matching the configuration on BackboneSW.

<img width="886" height="410" alt="image" src="https://github.com/user-attachments/assets/fafd3d4f-71ef-45a3-b0e5-cbefdf211db2" />

Interfaces `G1/0` and `G1/2` were configured as 802.1Q trunks towards the other switches.

The trunks use 802.1Q encapsulation, allow VLANs 10 and 20, and use VLAN 999 as the native VLAN.

VLAN 10 was also created locally. SW2 does not have any Office hosts, but the VLAN needs to exist on the switch so that VLAN 10 traffic can be carried across the trunk links.

<img width="667" height="120" alt="image" src="https://github.com/user-attachments/assets/94e78fb4-a26e-44c3-b3fb-f29b6ee90b5f" />

DTP was disabled on both trunk interfaces with `switchport nonegotiate`.

<img width="886" height="86" alt="image" src="https://github.com/user-attachments/assets/3e720f84-f31c-4391-9a4b-acbf30fe16d6" />

The configuration was saved after completing the switch setup.
