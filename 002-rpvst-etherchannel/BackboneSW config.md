# Backbone Switch Configuration

`BackboneSW` is the central switch of the topology. It acts as the STP root, provides the default gateways for both VLANs, terminates two EtherChannels, and provides the routed connections to R1 and the partner network.

<img width="541" height="87" alt="ba1" src="https://github.com/user-attachments/assets/f347b6cd-72bf-4c91-863f-dccd4f954d3b" />

The hostname was set to `BackboneSW` and privileged EXEC mode was protected with `enable secret`.

<img width="886" height="111" alt="image" src="https://github.com/user-attachments/assets/3741da5f-d023-4c1a-9932-46f5a514c3cb" />

Interfaces `G1/0–G1/3` were bundled into a static EtherChannel using `channel-group 1 mode on`. This creates `Port-channel 1`, which connects BackboneSW to SW1.

<img width="606" height="119" alt="image" src="https://github.com/user-attachments/assets/44906e60-7182-4487-b1f4-2f029fc759a3" />

VLAN 10 (`Office`) and VLAN 20 (`Industrial`) were created in the local VLAN database.

<img width="633" height="145" alt="image" src="https://github.com/user-attachments/assets/58d733d4-eeb0-43d3-863a-7388aeed1b02" />

`Port-channel 1` was configured as an access port in VLAN 10. The four physical links therefore operate as a single logical connection carrying Office traffic.

<img width="789" height="70" alt="image" src="https://github.com/user-attachments/assets/58681eaa-23aa-4fcb-bfa5-0d8f4f63bcdc" />

Rapid PVST+ was enabled with `spanning-tree mode rapid-pvst`.

<img width="825" height="178" alt="image" src="https://github.com/user-attachments/assets/df29755a-e14d-415d-b09d-eeef3fb3b6af" />

The EtherChannel load-balancing method was set to `src-dst-ip`. Root Guard was also enabled on `Port-channel 1` to prevent a neighboring switch from becoming the root bridge through this connection.

<img width="816" height="130" alt="image" src="https://github.com/user-attachments/assets/f8ab62f6-9030-4aac-aae9-92c271d9cccb" />

BackboneSW was configured as the primary root bridge for both VLANs:

```text
spanning-tree vlan 10 root primary
spanning-tree vlan 20 root primary
```

This makes the STP root selection explicit for the design.

<img width="886" height="248" alt="image" src="https://github.com/user-attachments/assets/d1b7cf36-9f83-469d-bb33-59732512d8e6" />

`G0/3` was converted into a routed interface with `no switchport` and assigned `10.60.0.49/30` for the point-to-point connection to `PARTNER-SW`.

<img width="886" height="128" alt="image" src="https://github.com/user-attachments/assets/80b3394e-f6f4-4d8c-b143-82816bc671f2" />

The VLAN 10 SVI was configured with `10.60.0.30/27`, the last usable address in the Office subnet. This address is used as the default gateway for the Office hosts.

![](images/bb-10-svi-vlan20.png)

The VLAN 20 SVI was configured with `10.60.0.46/28`, which is the last usable address in the Industrial Unit subnet and its default gateway.

<img width="886" height="116" alt="image" src="https://github.com/user-attachments/assets/bef74b28-5f77-490c-afe4-9afff6b0142b" />

`G2/0` was configured as a manual 802.1Q trunk towards SW3. The unused VLAN 999 was selected as the native VLAN, VLANs 10 and 20 were allowed, and DTP was disabled with `switchport nonegotiate`.

<img width="825" height="367" alt="image" src="https://github.com/user-attachments/assets/6f4b26b0-7ce2-4800-b7b6-e116c14d3df0" />

Root Guard was enabled on `G2/0` to prevent SW3 from becoming the root bridge through this connection.

<img width="653" height="52" alt="image" src="https://github.com/user-attachments/assets/99c661ff-4f6f-4886-b7e5-bad5e53c0bcb" />

Interfaces `G0/1–G0/2` were bundled into a second static EtherChannel using `channel-group 2 mode on`. This creates `Port-channel 2`, which connects BackboneSW to SW2.

<img width="741" height="145" alt="image" src="https://github.com/user-attachments/assets/a95f6050-412b-43bd-b6ff-6e997d66499a" />

`Port-channel 2` was configured as an access port in VLAN 20. The `src-dst-ip` load-balancing method is used for both EtherChannels.

<img width="886" height="176" alt="image" src="https://github.com/user-attachments/assets/441be8d1-14b6-4ae6-ad49-eb8a39c5f979" />

VTP was configured in transparent mode so that VLAN administration remains local to each switch.

`G0/0` was then converted into a routed interface and assigned `10.60.0.53/30` for the point-to-point connection to R1.

<img width="749" height="83" alt="image" src="https://github.com/user-attachments/assets/e075d6f9-c896-40c4-a715-11381220d6ed" />

A default static route was configured towards `10.60.0.54`, which is R1's address on the point-to-point link.

<img width="780" height="78" alt="image" src="https://github.com/user-attachments/assets/6a48ddc2-246a-4a14-8e4a-6ddede7921b6" />

The configuration was saved with `write memory`.

> **Note:** I forgot to configure Root Guard on `Port-channel 2` towards SW2. It does not affect the current lab, but it should be configured there as well to keep the STP protection consistent across all Layer 2 connections from BackboneSW.
