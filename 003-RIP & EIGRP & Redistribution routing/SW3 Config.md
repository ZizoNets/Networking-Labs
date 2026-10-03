# SW3 (Branch B Terrace) Configuration

`SW3` is the access switch of the Branch B terrace. It connects three PCs (`PC19`, `PC20` and `PC21`) and the uplink to `R3`. As in `SW2`, every port belongs to the same VLAN.

<img width="886" height="129" alt="image" src="https://github.com/user-attachments/assets/a5471f67-d138-4edb-a2ce-42d31f020d9a" />


The hostname was set to `SW3` and privileged EXEC mode was protected with `enable secret`.

<img width="886" height="301" alt="image" src="https://github.com/user-attachments/assets/4435cbc9-b3cf-4d6f-b015-b40b74fcdde2" />


VLAN 40 was created and named `Branch B Terrace`. Interfaces `G0/0–2` were then configured as access ports in VLAN 40 for the three terrace PCs.

<img width="886" height="195" alt="image" src="https://github.com/user-attachments/assets/b74aac95-b283-461e-acde-840e001440b7" />


`G0/3` was configured as an access port in VLAN 40. It is the connection to `Fa0/0` of `R3`, the gateway of the Terrace LAN.

During verification, `PC21` could not reach its gateway. The cause was a duplex mismatch between this port (full duplex) and `Fa0/0` of `R3`. The fix is described in the [R3 configuration](R3%20Config.md). The access port assignment of `G0/3` was also checked.

<img width="886" height="109" alt="image" src="https://github.com/user-attachments/assets/44dca637-c8b2-4f05-a058-b6a79790d687" />


VTP was also configured in transparent mode on `SW3` so VLAN configuration remains local to the switch. The screenshot of this command was not captured.

The configuration was saved with `write`.
