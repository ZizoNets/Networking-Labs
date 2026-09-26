# SW3 (Distribution) Configuration

`SW3` is the distribution switch in the topology. It has no directly connected end devices and is used to provide additional Layer 2 paths between `BackboneSW`, `SW1`, and `SW2`. All three connections are configured as trunks, making `SW3` part of the redundant Layer 2 design.

<img width="513" height="89" alt="image" src="https://github.com/user-attachments/assets/f16a7346-240e-4558-aef4-8c62655fa9fb" />

The hostname was set to `SW3` and privileged EXEC mode was protected with `enable secret`.

<img width="886" height="491" alt="image" src="https://github.com/user-attachments/assets/d78b0e56-9f69-4426-95ed-82e5f792944a" />

VLAN 10 (`Office`) and VLAN 20 (`Industrial`) were created locally. SW3 has no access ports in either VLAN, but both VLANs need to exist on the switch so they can be carried across its trunk links.

Interfaces `G0/0–G0/2` were configured as 802.1Q trunks, with one link towards each neighboring switch.

The first attempt to restrict the allowed VLANs used `switchport trunk vlan 10,20`, which IOS rejected as invalid syntax. The correct command is `switchport trunk allowed vlan 10,20`, which was then applied successfully.

<img width="625" height="56" alt="image" src="https://github.com/user-attachments/assets/451c3549-cc7c-4bff-8c55-e7b2aa6b7b10" />

`spanning-tree loopguard default` was enabled globally. This applies Loop Guard automatically to eligible non-designated point-to-point ports, instead of configuring it individually on specific interfaces as was done with the EtherChannels on `SW1` and `SW2`.

<img width="764" height="183" alt="image" src="https://github.com/user-attachments/assets/4b70b278-298b-4b6c-b615-811da2e1d368" />

VLAN 999 was configured as the native VLAN on the three trunk interfaces, and DTP was disabled with `switchport nonegotiate`.

<img width="678" height="56" alt="image" src="https://github.com/user-attachments/assets/dcd65f27-c212-4e16-b6eb-b099875a128e" />


VTP was configured in transparent mode.

The configuration was also saved with `write memory`. I did not capture a screenshot of this command, but it was executed successfully.
