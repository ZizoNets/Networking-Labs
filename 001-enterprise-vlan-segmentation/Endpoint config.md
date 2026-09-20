<img width="497" height="67" alt="Captura de pantalla 2026-09-20 124153" src="https://github.com/user-attachments/assets/12902289-3cf0-4016-9389-11a21c5c369f" />

Configured each end device with a static IPv4 address, the corresponding subnet mask, and the default gateway of its assigned VLAN. The default gateway is provided by the corresponding SVI on the Layer 3 switch.

<img width="885" height="687" alt="Captura de pantalla 2026-09-20 124407" src="https://github.com/user-attachments/assets/62e5faec-81da-4445-b82a-dcdfaefc4be4" />

Tested connectivity from the end devices by pinging their default gateway and randomly selected hosts across the other departmental VLANs. These tests verified local gateway reachability and confirmed that inter-VLAN routing was functioning correctly across the Layer 3 core switch, with the tested hosts reachable from each other.
