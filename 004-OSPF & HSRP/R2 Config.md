# R2 Configuration

`R2` is the second hotel router and mirrors `R1`. It routes VLANs 20 and 30 through subinterfaces on a single trunk towards `SW1`, has a direct link to the Guests VLAN on `SW2`, and connects to `R1` and `Backbone` with OSPF. It is the HSRP Active router for VLAN 10 (Guests).

<img width="886" height="130" alt="image" src="https://github.com/user-attachments/assets/85bd3207-a8d6-406c-a412-2aca3386d085" />


`G1/0` was enabled and configured with `10.40.0.30/27`. It connects directly to the Guests VLAN on `SW2`. The link carries a single VLAN, so it does not need subinterfaces, and `R2` needs an address there to route Guests traffic and take part in HSRP.

<img width="886" height="325" alt="image" src="https://github.com/user-attachments/assets/8c6edbad-37d8-4046-9741-502a3b5b7aa9" />


`G4/0` was configured with `10.40.0.58/29`, the shared segment with `R1` and `Backbone`. `G3/0` was configured with `10.40.0.65/30`, the direct link to `R1`, which gives the two routers a second path between them if the shared segment fails.

<img width="885" height="53" alt="image" src="https://github.com/user-attachments/assets/99859c97-be21-4d3d-be0c-5441f8d37383" />


`G5/0` is the trunk towards `SW1`. `SW1` sends two VLANs over that single cable and each VLAN needs its own gateway on the router, so one subinterface per VLAN was created (router-on-a-stick). `G5/0.30` was configured with `encapsulation dot1q 30` and `10.40.0.51/29` for Security.

<img width="885" height="29" alt="image" src="https://github.com/user-attachments/assets/0d0d2a7b-5bc2-4654-bc6d-eb5c65b1fcdf" />


`G5/0.20` was configured with `encapsulation dot1q 20` and `10.40.0.45/28` for Admin.

<img width="886" height="85" alt="image" src="https://github.com/user-attachments/assets/bd0c9b00-541c-4866-8243-28217292400e" />



A `G5/0.99` subinterface was also created, with `encapsulation dot1q 99 native`.



<img width="886" height="84" alt="image" src="https://github.com/user-attachments/assets/4335af8b-5cf7-4f08-a1cf-ecb1eb9b34df" />



`G5/0` was enabled.


<img width="886" height="100" alt="image" src="https://github.com/user-attachments/assets/621ea650-7552-4a31-b3e3-e67c208c3165" />


`Loopback2` was configured with `2.2.2.2/32`. It will be the router ID, since a loopback stays up as long as the router runs.

<img width="886" height="265" alt="image" src="https://github.com/user-attachments/assets/2bf59869-5214-4690-be8c-4144feed6eba" />


OSPF process 1 was started. The Guests network (`10.40.0.0/27`), the direct link to `R1` (`10.40.0.64/30`), the shared segment (`10.40.0.56/29`) and the loopback were added to Area 0. The `network` command enables OSPF on interfaces matching the specified range and advertises their networks.

The loopback was initially added using the subnet mask `255.255.255.255` instead of a wildcard mask. I later corrected the command to use the proper wildcard mask, `0.0.0.0`.

`Loopback2`, `G1/0` and `G5/0` were configured as passive interfaces, preventing `R2` from sending OSPF hello packets on the loopback and the VLANs connected to end devices, where no OSPF neighbors are expected.

After the first checks I added the two networks that were missing, plus the passive setting for the subinterfaces:

```text
R2(config)#router ospf 1
R2(config-router)#network 10.40.0.32 0.0.0.15 area 0
R2(config-router)#network 10.40.0.48 0.0.0.7 area 0
R2(config-router)#passive-interface g5/0.20
R2(config-router)#passive-interface g5/0.30
```
> **Note:** I corrected the missing OSPF `network` statements and configured the VLAN 20 and VLAN 30 subinterfaces as passive interfaces. These corrections are reflected in the final configuration, but I did not document them with screenshots.

<img width="853" height="25" alt="image" src="https://github.com/user-attachments/assets/ef4b1a1b-4e66-46d2-9592-67ca7638ec57" />


The reference bandwidth was changed to `10000`, matching `R1` and `Backbone`. With the default of 100 Mbps, Fast Ethernet and Gigabit both cost 1. With `10000` a Gigabit link costs 10, and it has to be the same on every router so the costs can be compared.

<img width="886" height="140" alt="image" src="https://github.com/user-attachments/assets/e5c14ffd-8a35-49d5-b2d4-3cfae249a3d6" />



HSRP was configured for VLAN 10 on `G1/0`: group 10, virtual IP `10.40.0.28`, priority 110 and `preempt`. The virtual IP is shared by `R1` and `R2` and is the gateway of the PCs, so if the Active router fails the other one answers for it and the PCs need no change. Priority 110 (the default is 100) makes `R2` the Active router, and `preempt` lets it take the role back after it recovers.

<img width="886" height="134" alt="image" src="https://github.com/user-attachments/assets/42adf396-f56c-42a1-a739-61ccb89b48e0" />



Groups 20 and 30 were configured on `G5/0.20` and `G5/0.30` with virtual IPs `10.40.0.44` and `10.40.0.53` and `preempt`. They keep the default priority 100, so `R2` is the Standby router for Admin and Security and `R1` (priority 110) is Active. Splitting the roles this way makes both routers forward traffic in normal operation.

<img width="775" height="122" alt="image" src="https://github.com/user-attachments/assets/7e19fce8-912d-4837-a307-f59f87c1c6c1" />



`ip ospf network point-to-point` was applied to `G3/0`. On Ethernet, OSPF would elect a DR and a BDR, which is pointless with only two routers, and point-to-point skips the election. `R1` has the same command on its end, because both ends must use the same network type.

<img width="886" height="144" alt="image" src="https://github.com/user-attachments/assets/959e3ec2-198b-4498-8aa9-a2a25842be0d" />


The configuration was saved with `write`.
