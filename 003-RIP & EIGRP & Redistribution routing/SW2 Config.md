# SW2 (Branch B Main) Configuration

`SW2` is the access switch of the Branch B main room. It connects ten PCs and the uplink to `R2`. Every port belongs to the same VLAN, so no trunk is needed.

<img width="878" height="141" alt="image" src="https://github.com/user-attachments/assets/c54c10ba-cbc5-4065-bad9-1718b7cd82d0" />


The hostname was set to `SW2` and privileged EXEC mode was protected with `enable secret`.

<img width="886" height="241" alt="image" src="https://github.com/user-attachments/assets/57823439-f653-4d5b-b40c-6dab5e031973" />


Interfaces `G0/0–3`, `G1/0–3` and `G2/0–1` were configured as access ports in VLAN 30. These ten ports are connected to the Branch B Main PCs.

VLAN 30 did not exist on the switch at this point, so IOS created it automatically when it was assigned to the interfaces. It was then named explicitly, first as `Branch B` and finally as `Branch B Main`.

<img width="886" height="173" alt="image" src="https://github.com/user-attachments/assets/d56eba19-1c58-4e7e-a3b7-c5e057751fdd" />


`G2/2` was configured as an access port in VLAN 30. It is the connection to `Fa0/0` of `R2`, the gateway of the Main LAN.
<img width="886" height="39" alt="image" src="https://github.com/user-attachments/assets/f8b6500a-c53a-4571-8dab-2960cbef733a" />


VTP was configured in transparent mode so VLAN configuration remains local to the switch.

<img width="886" height="98" alt="image" src="https://github.com/user-attachments/assets/f3f98674-8020-4ad5-9678-4deba2307d5a" />


The configuration was saved with `write`.
