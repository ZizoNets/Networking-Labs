# SW4 (Server Room) Configuration

`SW4` is the switch of the Server Room. It connects the four order servers and the uplink to `R4`. Every port belongs to VLAN 50, so no trunk is needed.

<img width="886" height="128" alt="image" src="https://github.com/user-attachments/assets/1477a52c-5182-4c2f-ba5a-2601207a3daa" />


The hostname was set to `SW4` and privileged EXEC mode was protected with `enable secret`.

<img width="702" height="195" alt="image" src="https://github.com/user-attachments/assets/fea87f6a-a06a-4a98-8006-b3a2f7227ca4" />


Interfaces `G0/0–3` and `G1/0` were configured as access ports in VLAN 50. Four of them are connected to the servers and the remaining one to `Fa3/0` of `R4`.

VLAN 50 did not exist on the switch at this point, so IOS created it automatically.

<img width="886" height="39" alt="image" src="https://github.com/user-attachments/assets/b2409fe8-69fa-4078-a8dd-240f1c6e565a" />


VLAN 50 was then named explicitly as `Server Room`.

<img width="886" height="59" alt="image" src="https://github.com/user-attachments/assets/6f16d5d4-0f5d-400d-9865-cbf6d5b1ed76" />


The configuration was saved with `write`.
