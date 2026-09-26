# End Device Configuration & Connectivity Verification

<img width="792" height="381" alt="image" src="https://github.com/user-attachments/assets/18e747ae-63bf-4b01-8425-11af22f14ea1" />

Each end device was configured with a static IPv4 address, subnet mask, and default gateway according to its VLAN.

`PC8`, one of the Office workstations, is shown here as an example. It was configured with `10.60.0.1/27` and `10.60.0.30` as its default gateway, which is the VLAN 10 SVI on `BackboneSW`.

The same configuration was applied to the remaining Office and Industrial Unit PCs, using addresses from their respective subnets.

<img width="822" height="188" alt="image" src="https://github.com/user-attachments/assets/8013248f-d695-4de1-87ad-b669854633de" />

Connectivity between the two VLANs was then tested. `PC1`, which belongs to VLAN 10, successfully pinged `10.60.0.33`, PC3_ address in the Industrial Unit subnet (VLAN 20).

All five ICMP packets were received successfully, with a TTL of 63. The TTL value is consistent with the traffic passing through the Layer 3 gateway on `BackboneSW`, rather than remaining within the same VLAN.

This confirms that inter-VLAN routing is working correctly and that the VLANs, trunk links, EtherChannels, STP configuration, and SVIs are operating as expected.


