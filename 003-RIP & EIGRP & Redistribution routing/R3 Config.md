# R3 (Branch B) Configuration

`R3` is the router of the Branch B terrace. It is the gateway for the Terrace LAN and one of the two routers serving Branch B. It has a GigabitEthernet link to `R4` and a serial link to `R2`, providing Branch B with two paths towards the rest of the network.

<img width="730" height="34" alt="image" src="https://github.com/user-attachments/assets/0b6b8587-d144-4e07-9b86-aed381368d92" />

Privileged EXEC mode was protected with `enable secret`.

<img width="886" height="138" alt="image" src="https://github.com/user-attachments/assets/308c4316-6833-4b8c-b975-a3543c9e166e" />

`S2/0` was configured with `192.168.90.10/30` and enabled with `no shutdown`. `R3` is the DTE side of this link, so the clock rate is provided by `R2`, and the interface came up without requiring a `clock rate` command.

The Terrace gateway address `10.90.0.30`, the last usable address of `10.90.0.16/28`, was initially configured on the wrong interface. The original topology also used a FastEthernet-to-GigabitEthernet connection, which caused a duplex mismatch during autonegotiation and prevented the link from operating correctly.

The connection was corrected by removing the original cable and replacing it with a GigabitEthernet-to-GigabitEthernet connection. The IP configuration was then moved to the GigabitEthernet interface connected to `SW3`:

```text
interface GigabitEthernet1/0
 ip address 10.90.0.30 255.255.255.240
 no shutdown
```

After correcting the physical connection and the interface IP configuration, the link came up successfully.

<img width="886" height="123" alt="image" src="https://github.com/user-attachments/assets/38a30666-1384-4957-be32-8dff156085d0" />

`G3/0` was configured with `192.168.90.13/30`, the routed link towards `R4`. The interface and line protocol came up successfully.

<img width="886" height="90" alt="image" src="https://github.com/user-attachments/assets/88c90021-ef5e-4399-b8f9-e6c89b78442b" />

`Loopback1` was created with `3.3.3.3/32` to identify the router. Because the loopback existed before the EIGRP process was configured, IOS selected it automatically as the router ID.

<img width="886" height="194" alt="image" src="https://github.com/user-attachments/assets/9b185761-dcb7-44f7-8439-c52541a453b8" />

EIGRP AS 1 was configured with the Terrace LAN and the two router-to-router links.

The GigabitEthernet interface connected to the Terrace LAN and `Loopback1` were configured as passive, so no EIGRP neighbors are formed on the LAN or loopback interface. The router-to-router links remain active for EIGRP neighbor formation, and `no auto-summary` was applied.

<img width="886" height="174" alt="image" src="https://github.com/user-attachments/assets/aa6b96e7-1ddb-46bb-85ce-e707e22ee699" />

The configuration was saved with `write`. IOS asked for confirmation because the previous NVRAM configuration had been written by a different version of the system image.

## Troubleshooting

### Duplex mismatch with SW3

`PC21` could not reach its gateway `10.90.0.30`. The console showed `%CDP-4-DUPLEX_MISMATCH` between `Fa0/0` of `R3` and `G0/3` of `SW3`.

The issue was caused by the original FastEthernet-to-GigabitEthernet connection, which resulted in a duplex mismatch during autonegotiation. Instead of forcing the duplex manually, the original cable was removed and the connection was replaced with a GigabitEthernet link. The IP configuration was then adjusted to the new interfaces, and the access port assignment of `G0/3` in VLAN 40 was verified on `SW3`.

After replacing the link and correcting the interface configuration, `PC21` was able to reach its gateway successfully.

### Wrong mask on the Terrace gateway

The routing tables captured during verification showed the Terrace network as `10.90.0.28/30` instead of the intended `10.90.0.16/28`. This meant that the LAN interface had the mask `255.255.255.252` at that moment.

With a `/30` mask, `R3` considered the gateway to belong to the `10.90.0.28/30` subnet rather than the intended `10.90.0.16/28` LAN. As a result, hosts such as `10.90.0.17` were not considered directly connected to the gateway.

The mask was corrected to `255.255.255.240` as shown above. The later successful pings to `10.90.0.17` from `PC1` and `Server1` confirmed that the Terrace LAN was reachable with the correct mask.
