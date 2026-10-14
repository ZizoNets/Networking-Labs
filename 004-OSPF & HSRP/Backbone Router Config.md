# Backbone Router Configuration

`Backbone` is the upstream router and simulates the provider. It shares the `10.40.0.56/29` segment with `R1` and `R2`, runs OSPF with them and sends them the default route.

<img width="886" height="149" alt="image" src="https://github.com/user-attachments/assets/cd9fc08e-0a8c-4b18-b337-c26ff08e3998" />


Privileged EXEC mode was protected with `enable secret`. `G1/0` was configured with `10.40.0.57/29` and enabled. It connects to `R1` and `R2` through `SW3`.

<img width="803" height="58" alt="image" src="https://github.com/user-attachments/assets/e0b3b951-db98-4581-bb63-d0eeff7a8d26" />


OSPF process 1 was started and the `10.40.0.56/29` network was added to Area 0. That turns on OSPF on `G1/0`, so `Backbone` can form adjacencies with both hotel routers.

<img width="886" height="82" alt="image" src="https://github.com/user-attachments/assets/22f1c827-6130-4209-a5ff-e6449a96fe2d" />


The reference bandwidth was changed to `10000`, the same value used on `R1` and `R2`. IOS warns that it has to be consistent on all routers, because OSPF costs are only comparable if every router calculates them the same way.

<img width="886" height="237" alt="image" src="https://github.com/user-attachments/assets/520e73b1-ad96-4b1d-b18d-799963cfd3f1" />


`Loopback3` was configured with `3.3.3.3/32`, advertised in Area 0 and set as passive. A loopback never goes down while the router is running, so OSPF uses it as a stable router ID without needing the `router-id` command. It is passive because it has no neighbors.

<img width="886" height="139" alt="image" src="https://github.com/user-attachments/assets/480e44d4-3701-4894-8005-dbfda5cec0fd" />

A static default route to Null0 was added, together with default-information originate in the OSPF process. With that, Backbone advertises 0.0.0.0/0 to R1 and R2, so they don't need a static default route of their own.


The WAN is only a mock-up and there is no real next hop. A route to `Null0` is always installed, so it can be advertised. Traffic to Internet addresses is simply discarded here, which means pings to them are expected to fail.

> **Note:** In my first version, I configured the default route to point to `1.1.1.1`, intending to simulate a Cloudflare server. I didn't realize that `1.1.1.1` was also the loopback address of `R1`, so the next hop resolved through OSPF to `R1` rather than to an external destination. I replaced it with a route to `Null0`, since this lab only tests OSPF default route propagation and does not simulate actual Internet connectivity.
<img width="886" height="142" alt="image" src="https://github.com/user-attachments/assets/ba8a3100-32f1-4e13-ab21-4ddf3775cfb4" />


The configuration was saved with `write`.
