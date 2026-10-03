# R5 (ISP) Configuration

`R5` simulates the ISP. It has a single serial link towards `R4` and only needs to be reachable at `192.168.90.18`, which is the next hop used by the default route configured on `R4`.

<img width="886" height="274" alt="image" src="https://github.com/user-attachments/assets/04396cd9-dc23-4030-89d4-b2b131ba3dac" />

`S2/0` was enabled with `no shutdown`. `R5` is the DTE side of the link, so the clock is provided by `R4` (`clock rate 128000`). The interface and line protocol came up as soon as `R4` was configured.

The address `192.168.90.18/30` was then configured on `S2/0` and the configuration was saved with `write`.

> **Note:** Only the ISP router's interface address was configured on `R5` so that `R4` could use `192.168.90.18` as the next hop for its default route. Since `R5` is only being used to simulate an ISP in this lab, no return routes towards the internal LANs were configured. The objective was to verify the default route towards the simulated ISP, not to provide full end-to-end Internet connectivity or responses from `R5` to the internal hosts.
