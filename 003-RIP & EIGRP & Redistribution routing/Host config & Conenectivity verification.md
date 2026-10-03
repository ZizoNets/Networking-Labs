# End Device Configuration & Connectivity Verification

<img width="886" height="167" alt="image" src="https://github.com/user-attachments/assets/81249e67-063b-43b2-a6c9-84de24b2aa68" />


Each end device was configured with a static IPv4 address, subnet mask and default gateway according to its VLAN.

`PC8`, one of the Branch A back-room PCs, is shown here as an example. It was configured with `10.90.0.41/29` and `10.90.0.46` as its default gateway, which is the `G1/0.20` subinterface of `R1`.

The same logic was applied to the other tested devices, using addresses from their own subnets and the last usable address as gateway.

## Routing tables

The routing tables were checked on the four routers after convergence.

<img width="886" height="634" alt="image" src="https://github.com/user-attachments/assets/6c9cda82-54d7-4ba9-871a-0df5fd1d2ded" />


`R1` has the connected Branch A networks on `G1/0.10` and `G1/0.20`. Everything else is learned through RIP via `192.168.90.2` (`R4`): Branch B, the Server Room, the loopbacks and the link networks. The gateway of last resort is `192.168.90.2` and the default route is marked `R*`, so it was originated by `R4` and not configured manually.

<img width="886" height="591" alt="image" src="https://github.com/user-attachments/assets/625edf4d-5efd-4de0-8970-7eb37ce8c92d" />


`R2` learns the Server Room and `R3`'s LAN through EIGRP via `192.168.90.6` (`R4`). Branch A (`10.90.0.32/29` and `10.90.0.40/29`) appears as `D EX`, which confirms that the RIP routes are redistributed into EIGRP. The default route is `D*EX 0.0.0.0/0` via `R4`.

<img width="886" height="561" alt="image" src="https://github.com/user-attachments/assets/8ccb7e90-8d66-4a09-b1b3-8a6ccbad4832" />


`R3` shows the mirrored result: EIGRP routes via `192.168.90.14` (`R4`), Branch A as `D EX` and the `D*EX` default route.

<img width="886" height="588" alt="image" src="https://github.com/user-attachments/assets/1e572430-715c-4cfe-815d-be63674aaf39" />


`R4` has the static default route (`S*`) towards `192.168.90.18`, RIP routes for Branch A via `192.168.90.1` and EIGRP routes for Branch B. The serial network `192.168.90.8/30` is reachable through two equal-cost EIGRP paths, one via `R3` and one via `R2`.

> **Note:** these captures were taken while `Fa0/0` of `R3` still had a `/30` mask, so the Terrace network appears as `10.90.0.28/30` instead of the planned `10.90.0.16/28`. The mask was corrected afterwards (see the [R3 configuration](R3%20Config.md)).

## End-to-end tests

Connectivity was tested from hosts in different domains. The TTL of each reply shows how many routers were crossed.

<img width="886" height="123" alt="image" src="https://github.com/user-attachments/assets/6c8d07d7-57f2-436f-ae50-39b1a325f504" />


`PC21` (Branch B Terrace) successfully pinged `10.90.0.49` in the Server Room. The TTL of 62 corresponds to two routers: `R3` and `R4`.

<img width="886" height="146" alt="image" src="https://github.com/user-attachments/assets/0896858d-2dba-46b8-87d1-d0f1ddb41a65" />


`PC21` pinged `10.90.0.41` (`PC8`, Branch A back room). The TTL of 61 corresponds to `R3`, `R4` and `R1`. This confirms that traffic crosses from the EIGRP domain to the RIP domain.

<img width="886" height="146" alt="image" src="https://github.com/user-attachments/assets/e5fac1cb-2a34-4974-9c2f-2a2739cc09be" />


`PC21` also pinged `10.90.0.33` in the Branch A Coffee room, with the same TTL of 61.

<img width="886" height="385" alt="image" src="https://github.com/user-attachments/assets/e598c419-1934-4024-92b0-873190e69b37" />


From `PC1` (Branch A Coffee, VLAN 10), three tests were run:

| Destination | Location               | TTL | Routers crossed |
| ----------- | ---------------------- | --: | --------------- |
| 10.90.0.41  | Branch A BR (VLAN 20)  |  63 | R1              |
| 10.90.0.17  | Branch B Terrace       |  61 | R1, R4, R3      |
| 10.90.0.49  | Server Room            |  62 | R1, R4          |

The first test confirms router-on-a-stick: the traffic between VLAN 10 and VLAN 20 goes through `R1` and the TTL is only reduced by one.

<img width="886" height="164" alt="image" src="https://github.com/user-attachments/assets/906dac79-4223-40a7-ba4a-a88b29c225ed" />


`Server1` pinged `10.90.0.17` in the Branch B Terrace with a TTL of 62 (`R4` and `R3`).

<img width="886" height="123" alt="image" src="https://github.com/user-attachments/assets/dd6331de-fd8d-45d2-b999-0b821f0e5717" />


`Server1` pinged `10.90.0.41` in Branch A with a TTL of 62 (`R4` and `R1`).

All the tests were successful. They confirm that Branch A and Branch B can reach each other and the Server Room through the redistribution configured on `R4`, and that the router-on-a-stick design works in Branch A.
