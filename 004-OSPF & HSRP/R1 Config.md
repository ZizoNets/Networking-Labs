# R1 Configuration

`R1` is one of the two hotel routers. It routes VLANs 20 and 30 through subinterfaces on a single trunk towards `SW1`, has a direct link to the Guests VLAN on `SW2`, and connects to `R2` and `Backbone` with OSPF. It is the HSRP Active router for VLANs 20 and 30.

<img width="886" height="70" alt="image" src="https://github.com/user-attachments/assets/a6c4cdec-f635-466c-810a-9be45a3d4c4c" />

Privileged EXEC mode was protected with `enable secret`, so nobody with console or remote access can change the configuration without the password. Unlike `enable password`, it is stored hashed.

<img width="886" height="297" alt="image" src="https://github.com/user-attachments/assets/99bfeaa6-25df-4cd8-8b37-58251ac4f46a" />

`G1/0`, the trunk towards `SW1`, was enabled. `SW1` sends both VLANs over a single cable and each VLAN needs its own gateway on the router, so one subinterface per VLAN was created (router-on-a-stick):

* `G1/0.2` with `encapsulation dot1q 20` and `10.40.0.46/28` for Admin.
* `G1/0.3` with `encapsulation dot1q 30` and `10.40.0.54/29` for Security.

The subinterface number does not need to match the VLAN ID, because the VLAN is defined by `encapsulation dot1q`.

<img width="886" height="73" alt="image" src="https://github.com/user-attachments/assets/be088d1e-0175-4065-82f4-b93d936c3f6a" />

`G4/0` was configured with `10.40.0.59/29`. It connects to the shared segment with `R2` and `Backbone` through `SW3`, which gives `R1` its path to `Backbone` and lets it form OSPF adjacencies there.

<img width="886" height="81" alt="image" src="https://github.com/user-attachments/assets/bc75a61e-05d7-41e5-9f8e-b2f22f5a225d" />

`G3/0` was configured with `10.40.0.66/30`. It is the direct link to `R2`, a second path between the routers that keeps them connected if the shared segment fails. A `/30` is enough because the link only needs two addresses.

<img width="886" height="110" alt="image" src="https://github.com/user-attachments/assets/06acad0d-8b4c-4e77-9aa4-6e35c50f858d" />

`G5/0` was configured with `10.40.0.29/27`. It connects directly to the Guests VLAN on `SW2`. The link carries a single VLAN, so it does not need subinterfaces, and `R1` needs an address there to route Guests traffic and take part in HSRP.

<img width="886" height="66" alt="image" src="https://github.com/user-attachments/assets/faa4dd65-1af9-42f2-9798-f6a63003c2c5" />

OSPF process 1 was started, and the Admin (`10.40.0.32/28`) and Security (`10.40.0.48/29`) networks were added to Area 0 with wildcard masks. The `network` command enables OSPF on the interfaces inside the range and advertises that subnet, and the wildcard matches the size of each subnet. One area is enough for a network this small.

<img width="886" height="43" alt="image" src="https://github.com/user-attachments/assets/537ce744-c117-44de-b2f5-0362a7f5ed3d" />

`G1/0.2` and `G1/0.3` were set as passive, so OSPF does not send hellos to the PCs. There are no routers on those VLANs, so the hellos would be useless. The networks are still advertised.

<img width="844" height="122" alt="image" src="https://github.com/user-attachments/assets/f57b6fea-a581-4886-ae96-1f475311c742" />

The shared segment (`10.40.0.56/29`), the direct link to `R2` (`10.40.0.64/30`) and the Guests network (`10.40.0.0/27`) were added to Area 0, so `R1` forms adjacencies with `R2` and `Backbone` and advertises the Guests network. `G5/0` was set as passive for the same reason as the other user-facing interfaces.

<img width="886" height="210" alt="image" src="https://github.com/user-attachments/assets/87341d12-1a5a-43c2-85e1-6c1ec99fb569" />

`Loopback1` was configured with `1.1.1.1/32`, advertised in Area 0 and set as passive. A loopback never goes down while the router is running, so OSPF uses it as a stable router ID and no `router-id` command was needed. It is passive because it has no neighbors.

<img width="816" height="31" alt="image" src="https://github.com/user-attachments/assets/c2a70570-253d-4956-a2c3-57af79828faf" />

The reference bandwidth was changed to `10000`, matching `R2` and `Backbone`. With the default of 100 Mbps, Fast Ethernet and Gigabit links both cost 1. With `10000`, a Gigabit link costs 10 and a Fast Ethernet link costs 100. It must be the same on all routers so the costs can be compared.

<img width="886" height="178" alt="image" src="https://github.com/user-attachments/assets/f4f5438d-fb25-44f4-8f3c-284e2349f32a" />

HSRP was configured for VLAN 20 on `G1/0.2`: group 20, virtual IP `10.40.0.44`, priority 110 and `preempt`. The virtual IP is shared by `R1` and `R2` and is the gateway of the PCs, so if the Active router fails the other one answers for it and the PCs need no change. Priority 110 (the default is 100) makes `R1` the Active router, and `preempt` lets it take the role back after it recovers.

The first attempt, `standby 20 10.40.0.44`, was rejected because the `ip` keyword was missing. The correct command is `standby 20 ip 10.40.0.44`.

<img width="886" height="123" alt="image" src="https://github.com/user-attachments/assets/864c577c-a39c-4c35-a043-ea76e4a1478b" />

The same was done for VLAN 30 on `G1/0.3`: group 30, virtual IP `10.40.0.53`, priority 110 and `preempt`. Security gets the same gateway redundancy as Admin, with `R1` as the preferred router.

<img width="886" height="116" alt="image" src="https://github.com/user-attachments/assets/ef75214c-e6ce-4eb9-bc53-56b40569b946" />

HSRP group 10 was configured on `G5/0` with virtual IP `10.40.0.28` and `preempt`. It keeps the default priority 100, so `R1` is the Standby router for Guests and `R2` (priority 110) is Active. Splitting the roles this way makes both routers forward traffic in normal operation.

<img width="691" height="52" alt="image" src="https://github.com/user-attachments/assets/113d2aef-cf82-4186-ae4b-39b85404a420" />

`ip ospf network point-to-point` was applied to `G3/0`. On Ethernet, OSPF would elect a DR and a BDR, which is pointless with only two routers, and point-to-point skips the election. `R2` has the same command on its end, because both ends must use the same network type.

<img width="886" height="145" alt="image" src="https://github.com/user-attachments/assets/ab3df273-4fe4-48d0-ae70-1f25adbe4786" />

The configuration was saved with `write`, because the running configuration is lost on reload otherwise. IOS warned that the NVRAM configuration had been written by a different version of the system image, and the overwrite was confirmed.
