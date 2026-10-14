# Verification

This page has the end device configuration, the `show` outputs of every device after the final configuration, the connectivity tests and the failover tests. Everything was taken with both routers up, except the failover tests.

## End Devices

One PC per VLAN was configured with a static IP address, subnet mask and default gateway. The gateway is the HSRP virtual IP of each VLAN, so the PCs don't depend on a single router.

<img width="886" height="87" alt="image" src="https://github.com/user-attachments/assets/60e4fc5f-942c-4c71-b887-2cc4835d66a0" />


`PC8` (Admin) was configured with `10.40.0.33/28` and the virtual IP `10.40.0.44` as gateway.

<img width="886" height="177" alt="image" src="https://github.com/user-attachments/assets/c9e6fb5d-65b1-4a22-a0d4-d54aa447504e" />


`PC1` (Guests) was configured with `10.40.0.1/27` and the virtual IP `10.40.0.28` as gateway.

<img width="886" height="164" alt="image" src="https://github.com/user-attachments/assets/e3d63858-a47d-44d7-91f8-15b3fd5c8551" />


`PC12` (Security) was configured with `10.40.0.49/29` and the virtual IP `10.40.0.53` as gateway. The configuration was saved on `PC1` and `PC12` with `save`.

## SW1

<img width="886" height="294" alt="image" src="https://github.com/user-attachments/assets/aace5a5e-bed6-42a2-80cf-ca0b66001e2e" />


`show vlan brief` shows VLAN 20 (`Admin`) on `G0/0–G0/2` and VLAN 30 (`Security`) on `G0/3` and `G1/0`. VLAN 99 exists but has no ports.

<img width="886" height="363" alt="image" src="https://github.com/user-attachments/assets/a74ca3ca-d27b-410d-8165-b603f9f8d338" />

`show interfaces status` shows the five access ports connected, each one in its VLAN, and `G1/1` and `G1/2` as trunks.

<img width="886" height="185" alt="image" src="https://github.com/user-attachments/assets/4eba64b9-d024-4db0-a167-ff303ebcb8e0" />


CDP shows `R1` on `G1/1` (its `Gig 1/0`) and `R2` on `G1/2` (its `Gig 5/0`), so each trunk goes to the router it was meant to.

<img width="886" height="347" alt="image" src="https://github.com/user-attachments/assets/df131794-6cee-47b0-9ffb-c94a990eea32" />


Both trunks are up with 802.1Q, and VLANs 20 and 30 are allowed and forwarding. The native VLAN mismatch between `G1/1` and `G1/2` was corrected, and both trunks are now configured with native VLAN 99.

## SW2

<img width="886" height="270" alt="image" src="https://github.com/user-attachments/assets/5ca073f5-5f9f-487f-9f75-61c8e581ad47" />


`show vlan brief` shows VLAN 10 (`Guests`) on `G0/0–G0/3`, `G1/0–G1/3` and `G2/0`. VLANs 20, 30 and 99 also appear in the VLAN database without ports. They aren't used on this switch.

<img width="886" height="368" alt="image" src="https://github.com/user-attachments/assets/0d6d4964-7454-468a-b3be-357c82d0871e" />


`show interfaces status` shows nine connected ports, all in VLAN 10. The rest are unused and stay in VLAN 1.

<img width="886" height="181" alt="image" src="https://github.com/user-attachments/assets/a186dd0b-f348-4469-8b4e-e98ba0db10ee" />


CDP shows `R2` on `G1/3` (its `Gig 1/0`) and `R1` on `G2/0` (its `Gig 5/0`).

## R1

<img width="886" height="140" alt="image" src="https://github.com/user-attachments/assets/a3d3dfa5-fefb-45c3-89da-e30b570b45f2" />


`show standby brief` shows `R1` Active for groups 20 and 30 with priority 110, and Standby for group 10, where `R2` (`10.40.0.30`) is the Active router. Preempt is on for all three groups.

<img width="886" height="106" alt="image" src="https://github.com/user-attachments/assets/6c1bf4fb-314c-423a-8656-4d6bd12d2205" />


`R1` has `FULL` adjacencies with `R2` (`2.2.2.2`) over `G3/0` and over `G4/0`, and with `Backbone` (`3.3.3.3`), which is the DR. The one on `G3/0` shows no role because it is a point-to-point link.

<img width="886" height="163" alt="image" src="https://github.com/user-attachments/assets/309d5e46-84e5-4991-8751-08c9ae2580aa" />


Six OSPF interfaces, all in Area 0. Every interface costs 10 except the loopback, which costs 1. `G3/0` is `P2P`, and on `G4/0` `R1` is `DROTH`.

<img width="886" height="594" alt="image" src="https://github.com/user-attachments/assets/3b3e17a6-864f-4d91-850d-ca4a1e9c68d2" />


`show ip protocols` shows router ID `1.1.1.1`, the six networks in Area 0, and `G1/0.2`, `G1/0.3`, `G5/0` and `Loopback1` as passive.

<img width="886" height="499" alt="image" src="https://github.com/user-attachments/assets/b745d05f-de0a-41fb-8ca3-ff4a4871394e" />


The routing table has the default route `O*E2 0.0.0.0/0` via `10.40.0.57` (`Backbone`). The loopback of `R2` has two equal-cost paths, one through `G3/0` and one through `G4/0`, and the loopback of `Backbone` is reached through `10.40.0.57`. The three VLANs and the two transit networks are directly connected.

## R2


<img width="886" height="156" alt="image" src="https://github.com/user-attachments/assets/e485a3a0-877d-45e6-8dfa-e7a3bf6e0830" />



`show standby brief` shows `R2` Active for group 10 with priority 110, and Standby for groups 20 and 30, where `R1` (`10.40.0.46` and `10.40.0.54`) is Active.


<img width="886" height="121" alt="image" src="https://github.com/user-attachments/assets/b8aa9c2e-1df1-40c6-b12a-adc77e1aed25" />


`R2` has `FULL` adjacencies with `R1` (`1.1.1.1`) over `G3/0` and over `G4/0`, and with `Backbone` (`3.3.3.3`) as DR.

<img width="886" height="118" alt="image" src="https://github.com/user-attachments/assets/a5891814-43b4-4162-8bfe-45e8d9d610a0" />




Router ID `2.2.2.2`, and the five networks plus the loopback in Area 0, including `10.40.0.32/28` and `10.40.0.48/29`. `G1/0`, `G5/0`, `G5/0.20`, `G5/0.30` and `Loopback2` are passive.

<img width="886" height="496" alt="image" src="https://github.com/user-attachments/assets/cc395549-a8e6-47d2-b8e2-ef43bc96d70c" />


Default route `O*E2 0.0.0.0/0` via `10.40.0.57`. The loopback of `R1` has two equal-cost paths, through `G3/0` and `G4/0`, and the loopback of `Backbone` is reached through `10.40.0.57`. The three VLANs and the two transit networks are directly connected.

## Backbone

<img width="886" height="92" alt="image" src="https://github.com/user-attachments/assets/cbe54c1c-f053-469b-8abd-cf981450c15d" />


`Lo3` and `Gi1/0` are in Area 0. `Gi1/0` costs 10, `Backbone` is the DR on it, and it has two neighbors.

<img width="886" height="91" alt="image" src="https://github.com/user-attachments/assets/54bfc53d-3392-48fb-96a2-fe355599fe2c" />


Both hotel routers are `FULL` neighbors: `R1` (`1.1.1.1`) as `DROTHER` and `R2` (`2.2.2.2`) as `BDR`. `Backbone` has the highest router ID and all priorities are the default, so it ended up as DR. DR election is not preemptive, so the result also depends on the order the routers came up.

<img width="886" height="589" alt="image" src="https://github.com/user-attachments/assets/69fb1ad1-730c-403f-9ba3-fe97481d6d78" />


Router ID `3.3.3.3`, two networks in Area 0 and `Loopback3` as passive. It also says `Backbone` is an autonomous system boundary router, because it originates the default route.

`show ip route` on `Backbone`:

```text
Gateway of last resort is 0.0.0.0 to network 0.0.0.0

S*    0.0.0.0/0 is directly connected, Null0
      1.0.0.0/32 is subnetted, 1 subnets
O        1.1.1.1 [110/11] via 10.40.0.59, 01:14:19, GigabitEthernet1/0
      2.0.0.0/32 is subnetted, 1 subnets
O        2.2.2.2 [110/11] via 10.40.0.58, 01:14:19, GigabitEthernet1/0
      3.0.0.0/32 is subnetted, 1 subnets
C        3.3.3.3 is directly connected, Loopback3
      10.0.0.0/8 is variably subnetted, 6 subnets, 5 masks
O        10.40.0.0/27 [110/20] via 10.40.0.59, 01:14:19, GigabitEthernet1/0
                      [110/20] via 10.40.0.58, 01:14:19, GigabitEthernet1/0
O        10.40.0.32/28 [110/20] via 10.40.0.59, 01:14:19, GigabitEthernet1/0
                       [110/20] via 10.40.0.58, 00:01:43, GigabitEthernet1/0
O        10.40.0.48/29 [110/20] via 10.40.0.59, 01:14:19, GigabitEthernet1/0
                       [110/20] via 10.40.0.58, 00:01:43, GigabitEthernet1/0
C        10.40.0.56/29 is directly connected, GigabitEthernet1/0
L        10.40.0.57/32 is directly connected, GigabitEthernet1/0
O        10.40.0.64/30 [110/20] via 10.40.0.59, 00:54:38, GigabitEthernet1/0
                       [110/20] via 10.40.0.58, 01:14:19, GigabitEthernet1/0
```

The default route is the only static route. Every user VLAN has two equal-cost paths, one through `R1` (`10.40.0.59`) and one through `R2` (`10.40.0.58`). The cost of 20 is the 10 of the shared segment plus the 10 of the VLAN interface on the router. The second path to VLANs 20 and 30 shows up with a recent timestamp (`00:01:43`) because it appeared once `R2` started advertising them.

## Connectivity

<img width="886" height="479" alt="image" src="https://github.com/user-attachments/assets/91399825-3311-45c7-82ff-61078a2449e4" />


`PC8` pinged the Guests virtual IP (`10.40.0.28`), a Security host (`10.40.0.49`) and a Guests host (`10.40.0.1`). All five ICMP packets were received in each test.

The replies from the two hosts have a TTL of 63. The PCs start at 64, so the replies crossed one router, which confirms that inter-VLAN routing works.

The reply from `10.40.0.28` has a TTL of 255, the starting TTL of a Cisco router, so it didn't cross any router on the way back. That fits with `R2` owning that virtual IP and also having its own subnet in VLAN 20: it can answer `PC8` directly, while `PC8` reaches it through `R1`, its gateway. So the way there and the way back are not the same path.

<img width="886" height="300" alt="image" src="https://github.com/user-attachments/assets/33e650c9-d35d-446d-8b34-9ed82dfd3525" />


`PC8` also reached the loopback of `R2` (`2.2.2.2`) and the loopback of `Backbone` (`3.3.3.3`). The reply from `2.2.2.2` has a TTL of 255 for the same reason as before, and the one from `3.3.3.3` has a TTL of 254, so it crossed one router on the way back.

Pings to Internet addresses are not expected to work, because the WAN is a mock-up and `Backbone` sends the default route to `Null0`.

## Failover

Both tests were done by powering off the router in GNS3. I didn't capture `show standby` during them, so the HSRP role changes are not shown directly. What the tests show is that traffic keeps flowing through the router that stays on.

### R1 powered off

<img width="886" height="503" alt="image" src="https://github.com/user-attachments/assets/f0753515-984f-4a5d-b13b-abe026ad6873" />


With `R1` off, `PC8` (Admin) pinged the Guests virtual IP, a Security host and the loopback of `Backbone`, and all of them replied. `R1` is the Active gateway for Admin, so `R2` must have taken over as gateway for VLAN 20. The TTL values match the previous test: 63 for the host that crossed a router, 255 for the address on `R2` itself and 254 for `Backbone`.

### R2 powered off

<img width="886" height="653" alt="image" src="https://github.com/user-attachments/assets/3c62a460-ad9d-4b87-8733-95e218eab50b" />


With `R2` off, `PC1` (Guests) pinged an Admin host (`10.40.0.33`), a Security host (`10.40.0.49`) and the loopback of `Backbone`, and all of them replied. `R2` is the Active gateway for Guests, so `R1` must have taken over as gateway for VLAN 10. The screenshot also shows the links of `R2` down in the topology.

## What Was Not Tested

* The HSRP failback with `preempt` (the router getting its Active role back) was not captured.
* Only one PC per VLAN was used in the tests.
