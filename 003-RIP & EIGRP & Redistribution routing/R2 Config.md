# R2 (Branch B) Configuration

`R2` is the router of the Branch B main room. It is the gateway of the Main LAN and forms EIGRP adjacencies with `R4` (Gigabit link) and `R3` (serial link), which gives Branch B two possible paths to the Server Room.

<img width="864" height="108" alt="image" src="https://github.com/user-attachments/assets/fa717b1a-7883-4bb4-8427-9838e02a0d75" />


Privileged EXEC mode was protected with `enable secret`.

<img width="886" height="161" alt="image" src="https://github.com/user-attachments/assets/8f14d221-f52d-45f2-8d3c-e3e6d12fdef3" />


`Fa0/0` was configured with `10.90.0.14/28`, the last usable address of `10.90.0.0/28` and the default gateway of the Main LAN. The interface was enabled with `no shutdown`.

<img width="886" height="122" alt="image" src="https://github.com/user-attachments/assets/d3962a33-0636-4b82-87cb-7634d781e996" />


`G1/0` was configured with `192.168.90.5/30`, the routed link towards `R4`.

<img width="886" height="475" alt="image" src="https://github.com/user-attachments/assets/09137566-cc6f-4242-aba2-e25c2d9bd776" />


The serial interface `S2/0` towards `R3` did not come up at first. `show controllers s2/0` shows that the cable is a V.11 (X.21) **DCE** cable, so `R2` is the side that must provide the clock.

<img width="886" height="299" alt="image" src="https://github.com/user-attachments/assets/c6a8204b-3999-415f-9ac3-110fb72ca18e" />


The valid values were listed with `clock rate ?` and `clock rate 128000` was applied. After `no shutdown`, the interface and the line protocol came up.

<img width="755" height="77" alt="image" src="https://github.com/user-attachments/assets/90f125b4-9ee9-4eb3-a52c-4d3ccc10fa38" />


`S2/0` was configured with `192.168.90.9/30`. `R3` uses `192.168.90.10/30` on the other side.

<img width="886" height="128" alt="image" src="https://github.com/user-attachments/assets/caf88752-9c68-4cf4-8f64-540012bc016a" />


EIGRP AS 1 was configured with `no auto-summary`. `Fa0/0` was set as passive, so no EIGRP neighbors are formed on the LAN. The two router-to-router links were added with wildcard masks:

```text
network 192.168.90.4 0.0.0.3
network 192.168.90.8 0.0.0.3
```

<img width="886" height="193" alt="image" src="https://github.com/user-attachments/assets/608955ce-efb7-4999-9534-d77e44a8948e" />


`Loopback2` was created with `2.2.2.2/32` to identify the router. It was set as passive and advertised in EIGRP. The Main LAN was also added with `network 10.90.0.0 0.0.0.15`.

<img width="886" height="496" alt="image" src="https://github.com/user-attachments/assets/551b855f-8936-493c-ae52-c5a1eb42a8a4" />


`show ip protocols` confirms AS 1, automatic summarization disabled and the advertised networks `2.2.2.2/32`, `10.90.0.0/28`, `192.168.90.4/30` and `192.168.90.8/30`. `Fa0/0` and `Loopback2` are passive.

The router ID was still `192.168.90.9` because the loopback was created after the EIGRP process had started, and a running process keeps its router ID.

<img width="886" height="58" alt="image" src="https://github.com/user-attachments/assets/15ad0104-2732-41a6-b8d2-0b8ca4f7366d" />


The router ID was set explicitly with `eigrp router-id 2.2.2.2`.

<img width="886" height="122" alt="image" src="https://github.com/user-attachments/assets/443013c7-2bcb-4da3-bc01-a2ffbad0b02d" />


The configuration was saved with `write memory`.
