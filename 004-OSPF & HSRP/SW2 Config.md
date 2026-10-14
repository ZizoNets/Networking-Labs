# SW2 (Guests) Configuration

`SW2` is the access switch for the Guests VLAN. It connects the seven Guests PCs and both hotel routers, which sit in VLAN 10 through plain access ports.

<img width="886" height="111" alt="image" src="https://github.com/user-attachments/assets/fa1fc6f2-e754-4cdf-ba5a-0ec0a83bbfb1" />


The hostname was set to `SW2` and privileged EXEC mode was protected with `enable secret`, so the configuration can't be changed without the password.

<img width="886" height="150" alt="image" src="https://github.com/user-attachments/assets/be3a0318-8296-42d9-becf-e82c020c23c6" />


`G0/0–G0/3` and `G1/0–G1/2` were set as access ports in VLAN 10 for the seven Guests PCs, and the VLAN was named `Guests`.





`G1/3`, the cable to `R2`, was also set as an access port in VLAN 10. The routers have an IP address directly on their physical interface for Guests, so for the switch they are just another device in the VLAN and no trunk is needed.

<img width="886" height="141" alt="image" src="https://github.com/user-attachments/assets/4fc5fdea-b48e-4820-baca-77be84d11356" />


`G2/0`, the cable to `R1`, was set up in the same way. I added this port later, after the first checks. I had forgotten it, so it stayed in VLAN 1, and `R1` and `R2` could not see each other in VLAN 10. Both ended up as HSRP Active for group 10 until the port was moved to VLAN 10.

<img width="886" height="99" alt="image" src="https://github.com/user-attachments/assets/44be331a-29c6-42dd-83e4-073b74766d68" />


The configuration was saved with `write`.
