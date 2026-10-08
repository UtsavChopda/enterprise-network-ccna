# Exact cabling

IOSvL2 interfaces use Gi0/0–Gi0/3, then Gi1/0–Gi1/3. Confirm with `show ip interface brief` before pasting.

| Device | Port | Peer | Peer port |
|---|---|---|---|
| HQ-R1 | Gi0/0 | ISP-R1 | Gi0/0 |
| BR-R1 | Gi0/0 | ISP-R1 | Gi0/1 |
| ISP-R1 | Gi0/2 | EXT-PC | eth0 |
| HQ-R1 | Gi0/1 | HQ-DSW1 | Gi0/0 |
| HQ-R1 | Gi0/2 | HQ-DSW2 | Gi0/0 |
| HQ-DSW1 | Gi0/1 | HQ-DSW2 | Gi0/1 |
| HQ-DSW1 | Gi0/2 | HQ-DSW2 | Gi0/2 |
| HQ-DSW1 | Gi0/3 | HQ-ASW1 | Gi0/0 |
| HQ-DSW2 | Gi0/3 | HQ-ASW1 | Gi0/1 |
| BR-R1 | Gi0/1 | BR-PC | eth0 |
| HQ-ASW1 | Gi0/2 | HR-PC | eth0 |
| HQ-ASW1 | Gi0/3 | FIN-PC | eth0 |
| HQ-ASW1 | Gi1/0 | IT-PC | eth0 |
| HQ-ASW1 | Gi1/1 | GUEST-PC | eth0 |
| HQ-ASW1 | Gi1/2 | SERVER-PC | eth0 |
