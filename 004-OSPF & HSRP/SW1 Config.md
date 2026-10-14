# SW1 (Admin & Security) Configuration

`SW1` is the access switch for Administration (VLAN 20) and Security (VLAN 30). The PCs connect to it through access ports, and it has one trunk to each hotel router.

<img width="886" height="56" alt="image" src="https://github.com/user-attachments/assets/b7deca21-13a3-4fe9-9b03-e48090f18751" />


The hostname was set to `SW1` so the device is easy to recognize, and privileged EXEC mode was protected with `enable secret` so nobody can change the configuration without the password.

<img width="886" height="161" alt="image" src="https://github.com/user-attachments/assets/df8aa0af-f3de-4648-bdb5-42777e196e2d" />


`G0/0–G0/2` were set as access ports in VLAN 20 for the Administration PCs. An access port keeps the PC in a single VLAN and sends its traffic untagged. The VLAN did not exist yet, so IOS created it automatically when it was assigned, and it was named `Admin` afterwards to make it easy to identify.

<img width="803" height="216" alt="image" src="https://github.com/user-attachments/assets/ef268252-14de-40de-8d41-babedc4c667e" />


`G0/3` and `G1/0` were set as access ports in VLAN 30 for the Security PCs. IOS created the VLAN in the same way, and it was named `Security`.

<img width="886" height="264" alt="image" src="https://github.com/user-attachments/assets/3898d7f5-5ce0-4414-b61a-b1de0c52ea34" />


VLAN 99 was created with the name `native`, and `G1/1`, the link towards `R1`, was turned into a trunk. It has to be a trunk because the link carries two VLANs over a single cable. The 802.1Q encapsulation was set explicitly with `switchport trunk encapsulation dot1q` before running `switchport mode trunk`, because the switch refuses trunk mode while the encapsulation is still on `auto`.

<img width="886" height="35" alt="image" src="https://github.com/user-attachments/assets/e901724d-e690-4b5d-9025-13a2f9cef8fb" />


Only VLANs 20 and 30 were allowed on the trunk, so no unused VLAN travels over the link.
<img width="886" height="191" alt="image" src="https://github.com/user-attachments/assets/5ba07150-c1e9-48f8-b3d5-4081d33530c9" />


`G1/2`, the link towards `R2`, got the same configuration: 802.1Q trunk with VLANs 20 and 30 allowed. Having a trunk to each router is what lets either one act as the gateway of these VLANs.

<img width="886" height="72" alt="image" src="https://github.com/user-attachments/assets/bfe8f8e5-8083-4318-bf18-a7c60f78ef58" />


The configuration was saved with `write`, so it survives a reload.
> **Note:** I corrected the native VLAN configuration so that both trunks use native VLAN 99. However, I did not document this correction with screenshots.
