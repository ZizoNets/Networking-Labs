# L2 Switch Configuration

The following configuration demonstrates the setup of `SW2`, which is the access switch assigned to the Sales department and VLAN 10.

The same configuration process is applied to the remaining Layer 2 switches, with the appropriate hostname, interfaces, and department VLAN adjusted according to the network design.

<img width="706" height="78" alt="Imagen3" src="https://github.com/user-attachments/assets/ccc9a326-2479-4e63-9516-084ded84da10" />

First, we configured the device hostname as `SW2` and secured privileged EXEC mode using `enable secret` to provide clear device identification and protect privileged administrative access.

<img width="616" height="278" alt="Imagen4" src="https://github.com/user-attachments/assets/2076f155-6b33-4095-a9aa-42c56909be0d" />

Next, we reviewed the initial port allocation to identify the interfaces that would be assigned to end devices and those that were not in use.

<img width="544" height="74" alt="Imagen7" src="https://github.com/user-attachments/assets/d7b0b254-1c43-4aae-bbe4-224c096da201" />

Created VLAN 10 in the local VLAN database and named it `Sales` to provide the required Layer 2 segmentation for the department.

<img width="559" height="81" alt="Imagen5" src="https://github.com/user-attachments/assets/02a74351-a1cf-4edf-84f0-dbc4ad81ffc5" />

Configured the selected switchports as static access ports and assigned them to VLAN 10. This ensures that end-user devices belong exclusively to the Sales broadcast domain and that the ports operate as access ports rather than trunks.

<img width="560" height="94" alt="Imagen8" src="https://github.com/user-attachments/assets/95e6bd19-3eab-4798-a47d-8d7e31b678dd" />

Set VTP to transparent mode using `vtp mode transparent`, keeping VLAN administration local to the switch and preventing VLAN changes from being propagated through VTP.

<img width="567" height="196" alt="Imagen9" src="https://github.com/user-attachments/assets/9e1017c8-427c-4cd7-aefb-4de4a2ff706a" />

Administratively shut down unused ports (Gi2/2–3, Gi3/0–3) to reduce the risk of unauthorized devices being connected to unused network interfaces.

<img width="508" height="60" alt="Imagen10" src="https://github.com/user-attachments/assets/2c7a8b6c-ef9b-4439-8c65-0bd2c3d00e43" />

Finally, saved the running configuration to persistent memory using `write memory` to ensure that the Layer 2 configuration is preserved after a device reboot.
