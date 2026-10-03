# SW1 (Branch A) Configuration

`SW1` is the only switch of Branch A. It connects the six PCs of the Coffee room (VLAN 10) and the two PCs of the back room (VLAN 20), and carries both VLANs to `R1` through a single 802.1Q trunk (router-on-a-stick).

<img width="886" height="124" alt="image" src="https://github.com/user-attachments/assets/f54c9319-6319-4424-9827-43bd21eeec7f" />


The hostname was set to `SW1` and privileged EXEC mode was protected with `enable secret`.

<img width="855" height="166" alt="image" src="https://github.com/user-attachments/assets/2f97783f-6470-459f-9d7d-388e0e75c62d" />


VLAN 10 (`Branch A Coffee`) and VLAN 20 (`Branch A BR`) were created in the local VLAN database.

<img width="886" height="244" alt="image" src="https://github.com/user-attachments/assets/d30c0e7f-3bbc-4868-9afa-632fe3c58917" />


Interfaces `G0/0–3` and `G1/0–1` were configured as access ports in VLAN 10. These six ports are connected to the Coffee room PCs.

Interfaces `G1/3` and `G2/0` were configured as access ports in VLAN 20 for the two back-room PCs.

<img width="886" height="152" alt="image" src="https://github.com/user-attachments/assets/66adbf6f-14ff-48ac-9e5d-906e6ec3fa5c" />


The remaining port, `G1/2`, was configured as the 802.1Q trunk towards `R1`. The encapsulation has to be set explicitly with `switchport trunk encapsulation dot1q` before trunk mode can be enabled.

The trunk only allows VLANs 10 and 20 and uses the unused VLAN 999 as native VLAN. DTP was disabled with `switchport nonegotiate`.

<img width="799" height="64" alt="image" src="https://github.com/user-attachments/assets/a1c25773-3423-40ab-9e52-7c9a428d511d" />


VTP was configured in transparent mode so VLAN configuration remains local to the switch.

<img width="808" height="77" alt="image" src="https://github.com/user-attachments/assets/0e3ae3ee-f9d3-4a4a-b22b-9f8ea4e8dccb" />


The configuration was saved with `write`.
