<img width="363" height="54" alt="Imagen1" src="https://github.com/user-attachments/assets/39fa2c09-eb1c-4d36-87a7-366dac660ac0" />

Configured the hostname as `L3SW`, secured privileged EXEC mode with `enable secret`, and enabled Layer 3 packet forwarding with `ip routing` to provide inter-VLAN routing.

<img width="567" height="170" alt="Imagen2" src="https://github.com/user-attachments/assets/47f1cc52-fab2-471d-9ef7-8d774f7e867a" />

Created VLANs `10, 20, 30, 40, 50, and 60` and assigned departmental names to provide logical network segmentation and separate broadcast domains.

<img width="542" height="76" alt="Imagen3" src="https://github.com/user-attachments/assets/16c56ecb-163e-46c6-8f49-0b639b519adf" />

Converted interface `Gi1/1` into a routed Layer 3 interface using `no switchport` and assigned `172.20.10.106/30` to establish the point-to-point transit link toward perimeter router `R1`.

<img width="567" height="51" alt="Imagen4" src="https://github.com/user-attachments/assets/7615fd3e-b662-4ebf-b5c2-fa59126c3c79" />

Created and enabled an SVI for each VLAN, assigning the last usable IP address of each subnet to provide the default gateway for the corresponding department.

<img width="466" height="44" alt="Imagen5" src="https://github.com/user-attachments/assets/67677d76-c97f-41be-bd54-561ab19f8695" />

Configured VTP transparent mode on the multilayer core switch to maintain local VLAN administration and prevent VLAN information from being propagated through VTP.

<img width="523" height="69" alt="Imagen6" src="https://github.com/user-attachments/assets/9b63b5df-3b77-4c62-afcf-7211ab064382" />

Configured an IPv4 default static route (`0.0.0.0/0`) pointing to R1's transit IP `172.20.10.105` to forward traffic destined for external networks toward the perimeter router.

<img width="567" height="66" alt="Imagen7" src="https://github.com/user-attachments/assets/805d57f0-7bac-4cb0-9174-8a3f7dbb25cb" />

Saved the running configuration to NVRAM using `write memory` to ensure the configuration persists after a system reload.
