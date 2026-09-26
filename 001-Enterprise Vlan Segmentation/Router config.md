<img width="459" height="49" alt="Imagen11" src="https://github.com/user-attachments/assets/7aad084d-6e6e-4cb6-863b-73e59341c492" />

Secured privileged mode access on perimeter router `R1` using `enable secret` to restrict privileged configuration access to authorized staff.

<img width="467" height="45" alt="Imagen12" src="https://github.com/user-attachments/assets/f45f2310-d630-46ef-b4c0-a327ded97036" />

Assigned IP address `172.20.10.105/30` to interface `F1/0` to establish the point-to-point transit link toward the core Layer 3 switch while optimizing IPv4 address utilization.

<img width="567" height="40" alt="Imagen13" src="https://github.com/user-attachments/assets/9e4a130a-4a58-48d6-821a-917b9e75400a" />

Activated the routed transit interface using the `no shutdown` command.

<img width="564" height="99" alt="Imagen15" src="https://github.com/user-attachments/assets/0f3de348-95f1-4cc3-9bf8-d4307515bf7d" />

<img width="411" height="66" alt="Imagen14" src="https://github.com/user-attachments/assets/75c5f1e4-86cc-4740-9b71-733c8a26cb18" />

Administratively disabled unused interfaces `FastEthernet0/1` and `F1/1–F1/7` to reduce the router's physical attack surface and prevent unauthorized connections through unused ports.

<img width="572" height="17" alt="Captura de pantalla 2026-09-20 122843" src="https://github.com/user-attachments/assets/17f7bd5b-69ea-4b7e-8038-c03d6dd512b5" />

Configured a summarized static route on `R1` for the internal `172.20.10.0/24` network via next-hop `172.20.10.106` (L3-SW), providing return-path reachability to all internal departmental VLAN subnets.

<img width="265" height="51" alt="Imagen16" src="https://github.com/user-attachments/assets/9fde6213-5d54-4e2e-8451-b3b470d8fb8a" />

Saved the running configuration to NVRAM using `write memory` to ensure the router configuration persists after a system reload.
