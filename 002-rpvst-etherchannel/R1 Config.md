# Router Configuration

`R1` acts as the perimeter router for the site. Its main role in this lab is to provide the routed connection from `BackboneSW` towards the network outside the site.

<img width="845" height="72" alt="image" src="https://github.com/user-attachments/assets/3dfa9f65-a3ae-4c36-ad6d-bfc6be0216a4" />

Privileged EXEC mode was protected with `enable secret`.

<img width="800" height="100" alt="image" src="https://github.com/user-attachments/assets/fd5f18c0-47da-4bc7-97e9-e62aa8b62c88" />

Interface `G1/0` was configured with `10.60.0.54/30`, which is the point-to-point link towards `BackboneSW`.
<img width="731" height="59" alt="image" src="https://github.com/user-attachments/assets/b1a1637e-0d65-4d2e-93f2-816e9fb87c1c" />


A static route was also configured for the internal `10.60.0.0/26` network, using `BackboneSW` at `10.60.0.53` as the next hop. This allows R1 to send return traffic back towards the internal VLANs.



The configuration was saved with `write memory`.
