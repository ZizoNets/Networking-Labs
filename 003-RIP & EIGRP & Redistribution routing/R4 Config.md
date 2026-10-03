# R4 (Server Room) Configuration

`R4` is the border router of the topology. It is the only device that speaks both protocols: RIPv2 towards `R1` and EIGRP towards `R2` and `R3`. It also hosts the Server Room gateway, redistributes between both domains and originates the default route towards the ISP.

<img width="817" height="34" alt="image" src="https://github.com/user-attachments/assets/eea0478f-e4ba-43e0-ab96-1e45c13205bf" />


Privileged EXEC mode was protected with `enable secret`.

## Interfaces

<img width="886" height="112" alt="image" src="https://github.com/user-attachments/assets/30d63e68-fecb-407f-b4cc-f090018ef94a" />


`G5/0` was configured with `192.168.90.2/30`, the routed link towards `R1`.

<img width="886" height="147" alt="image" src="https://github.com/user-attachments/assets/94677607-97e7-4dfe-a4ac-8ebed7fb9edc" />


`G4/0` was configured with `192.168.90.6/30`, the routed link towards `R2`.

<img width="886" height="139" alt="image" src="https://github.com/user-attachments/assets/aff9836e-a40a-487b-8db5-bce42d732649" />


`G1/0` was configured with `192.168.90.14/30`, the routed link towards `R3`.

<img width="886" height="134" alt="image" src="https://github.com/user-attachments/assets/66d52d60-a02d-4ca8-9e25-d638e02b2027" />


`Fa3/0` was configured with `10.90.0.54/29`, the last usable address of `10.90.0.48/29` and the default gateway of the Server Room.

<img width="886" height="242" alt="image" src="https://github.com/user-attachments/assets/1b238d56-ebbf-40be-becc-d92022f44984" />


The serial interface `S2/0` towards the ISP was down. `show controllers s2/0` shows a V.11 (X.21) **DCE** cable, so `R4` must provide the clock for this link.

<img width="886" height="199" alt="image" src="https://github.com/user-attachments/assets/a06496ba-317c-4ad7-868a-3e42da781de8" />


`clock rate 128000` was applied to `S2/0`, followed by `192.168.90.17/30` and `no shutdown`. The interface and the line protocol came up.

## EIGRP

<img width="886" height="171" alt="image" src="https://github.com/user-attachments/assets/c803bd2b-b47e-4502-9460-accc26d57393" />


EIGRP AS 1 was configured with `no auto-summary` and the following networks:

```text
network 10.90.0.48 0.0.0.7
network 192.168.90.4 0.0.0.3
network 192.168.90.12 0.0.0.3
```

`Fa3/0` was set as passive, so no neighbors are formed on the Server Room LAN. The links towards `R1` and the ISP are not part of EIGRP.

RIP routes were redistributed into EIGRP with a seed metric:

```text
redistribute rip metric 10000 100 255 1 1500
```

The values are bandwidth in kbps, delay in tens of microseconds, reliability, load and MTU. They represent a modest 10 Mbps path with 1000 µs of delay: worse than the Gigabit links of the EIGRP domain. The serial links only received a `clock rate`, so EIGRP still calculates them with the default bandwidth of `1544 kbps`. The routes appear in `R2` and `R3` as external routes (`D EX`).

## RIP

<img width="886" height="121" alt="image" src="https://github.com/user-attachments/assets/b38e0421-a49c-4114-a345-bd2ecbd755aa" />


RIP version 2 was enabled with `no auto-summary` and `network 192.168.90.0`.

EIGRP routes were redistributed into RIP with:

```text
redistribute eigrp 1 metric 1
```

RIP only uses hop count, and all the imported routes enter the RIP domain through `R4`, so a seed metric of `1` is sufficient. With this command, `R1` learns the Branch B, Server Room and link networks.

> **Note:** `network 192.168.90.0` initially enabled RIP on all four `192.168.90.x` interfaces of `R4`, including the links towards `R2`, `R3` and the ISP, where no RIP neighbor exists. After noticing this, the unnecessary interfaces were configured as passive:
>
> ```text
> router rip
>  passive-interface GigabitEthernet4/0
>  passive-interface GigabitEthernet1/0
>  passive-interface Serial2/0
> ```
>
> This prevents RIP updates from being sent through those interfaces while still allowing their connected networks to be advertised. No screenshot was captured after making this adjustment.

## Default route

<img width="834" height="164" alt="image" src="https://github.com/user-attachments/assets/639da2ce-8211-4f06-b278-7f3e36007be9" />


A static default route was configured towards `R5`, the simulated ISP:

```text
ip route 0.0.0.0 0.0.0.0 192.168.90.18
```

`default-information originate` was then added under `router rip`, so `R1` receives the default route through RIP.

<img width="886" height="48" alt="image" src="https://github.com/user-attachments/assets/30f62efa-d66b-4d1f-a8d7-613cf492813a" />


The static route was also redistributed into EIGRP with the same seed metric used for RIP routes:

```text
redistribute static metric 10000 100 255 1 1500
```

`R2` and `R3` receive the default route as an external EIGRP route (`D*EX`). No default routes are configured manually on the café routers.

<img width="886" height="104" alt="image" src="https://github.com/user-attachments/assets/8cdf2848-6435-440d-935b-1407bf527ae0" />


The configuration was saved with `write memory`. IOS asked for confirmation because the previous NVRAM configuration was written by a different version of the system image.
