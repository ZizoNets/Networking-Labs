# R1 (Branch A) Configuration

`R1` is the only router of Branch A. It is the default gateway of both VLANs through router-on-a-stick and the only RIPv2 speaker of the lab, with its link to `R4` as the only way out.

<img width="886" height="80" alt="image" src="https://github.com/user-attachments/assets/56777e40-c93c-453c-9583-7879fa629f0d" />


Privileged EXEC mode was protected with `enable secret`.

<img width="880" height="202" alt="image" src="https://github.com/user-attachments/assets/f7c26ac5-3800-4c15-b7c7-9de9b274cc16" />


Subinterface `G1/0.10` was created for VLAN 10 with `encapsulation dot1q 10` and the address `10.90.0.38/29`, the last usable address of the Coffee subnet and its default gateway.

<img width="886" height="153" alt="image" src="https://github.com/user-attachments/assets/70aff430-0005-49b6-a6de-2c1c4ae2e1b6" />


Subinterface `G1/0.20` was created for VLAN 20 with `encapsulation dot1Q 20` and the address `10.90.0.46/29`, the default gateway of the back room.

<img width="706" height="80" alt="image" src="https://github.com/user-attachments/assets/3b74f8af-2242-4636-b19f-ca49f0d40a97" />


The physical interface `G1/0` has no IP address. It was enabled with `no shutdown`, which brings up both subinterfaces.

<img width="886" height="64" alt="image" src="https://github.com/user-attachments/assets/4b4a66a4-4888-41d1-b3d5-c04848c950f2" />


`G4/0` was configured with `192.168.90.1/30`, the routed link towards `R4`.

This interface was administratively down by default. After applying `no shutdown`, the link came up and the RIP tables converged immediately.

<img width="886" height="167" alt="image" src="https://github.com/user-attachments/assets/64ca3123-dc13-482e-82aa-619f7006a84d" />


RIP version 2 was enabled with automatic summarization disabled. The `10.0.0.0` network covers the two LAN subinterfaces and `192.168.90.0` covers the link to `R4`.

The LAN interface `G1/0` was configured as passive so no RIP updates are sent towards the users.

<img width="886" height="151" alt="image" src="https://github.com/user-attachments/assets/e7cb7388-915e-4226-bfeb-007d2e9d5618" />


The RIP section of the running configuration was checked with `show running-config | section router rip`.

> **Note:** passive state is configured per interface. The passive command was applied to the physical `G1/0`, while the LANs use the subinterfaces `G1/0.10` and `G1/0.20`. It should be verified with `show ip protocols` that these subinterfaces do not send updates, or they should be added explicitly with `passive-interface G1/0.10` and `passive-interface G1/0.20`.

<img width="886" height="116" alt="image" src="https://github.com/user-attachments/assets/17e9ea60-a15a-4d4c-9ece-2af2da36646a" />


The configuration was saved with `write`. IOS warned that the previous NVRAM configuration was written by a different version of the system image and asked for confirmation.

`R1` has no static or default route. The default route is learned from `R4` through RIP.
