# Enterprise Network Design and Implementation

A multi-site networking project in EVE-NG

Utsav Chopda
Technical Project Report | Version 1.0 | 8 October 2026

## Executive summary

This project implements an isolated enterprise network connecting a Bengaluru headquarters and a Pune branch through a simulated service provider. The lab contains 13 nodes and 15 Ethernet links: three routers, three switches and seven lightweight endpoints. It brings switching, routing, address services, access control and gateway redundancy into one working design.

The headquarters network separates HR, Finance, IT, guest, server and management traffic into individual VLANs. OSPF provides site connectivity, DHCP supplies user addressing, and port address translation supports access to an external test segment. Guest access is restricted at the routed boundary. HSRP, Rapid PVST+ and LACP address different failure conditions at the headquarters.

The project demonstrates how these features depend on one another. VLAN membership and trunks establish the Layer 2 path; gateway interfaces and routes provide reachability; access rules determine which traffic may pass. Functional verification covers these relationships and the behavior of selected link and gateway failures.

The scope is a virtual networking lab. It does not include application hosting, internet service, a stateful firewall or production availability guarantees. The design retains several single points of failure, which are identified alongside the controls that are implemented.

### Report guide

| Section | Subject | Page |
| --- | --- | --- |
| 1 | Project scope and architecture | 2 |
| 2 | Addressing and segmentation | 3 |
| 3 | Routing and network services | 4 |
| 4 | Security and availability | 5 |
| 5 | Functional verification | 6 |
| 6 | Limitations and conclusion | 7 |
| Appendix A | Interface schedule | 8 |

Supporting materials: EVE-NG topology archive, per-device configuration files, detailed configuration procedures and an evidence register.


---

## 1 Project scope and architecture

The scenario represents a small enterprise with departmental networks at headquarters and a single branch LAN. The design provides routed communication between the sites and a controlled path to an external test endpoint. All links remain inside EVE-NG.

![Headquarters and branch topology](../Topology.svg)

At headquarters, two distribution switches provide VLAN gateways and a single access switch connects the test clients. The branch client connects directly to BR-R1. ISP-R1 acts as both the private transit network and the boundary of the external simulator; this is a teaching simplification, rather than a public ISP deployment model.

The host has 16 GB RAM, with 8 GB assigned to EVE-NG. Each IOSv or IOSvL2 node is allocated 1024 MB and one virtual CPU. The six network nodes therefore reserve 6144 MB before emulator overhead. VPCS endpoints keep the remaining load small. Application VMs are outside the baseline resource plan.


---

## 2 Addressing and segmentation

Private address space is allocated by location and function. Headquarters VLAN IDs are reflected in the third octet to simplify fault isolation. Point-to-point links use /30 prefixes and router loopbacks use /32 prefixes. The private ranges follow RFC 1918 [1]; the external simulator uses the documentation block 203.0.113.0/24 [2].

| VLAN | Function | Subnet | Virtual gateway |
| --- | --- | --- | --- |
| 10 | HR | 10.10.10.0/24 | 10.10.10.1 |
| 20 | Finance | 10.10.20.0/24 | 10.10.20.1 |
| 30 | IT | 10.10.30.0/24 | 10.10.30.1 |
| 40 | Guest | 10.10.40.0/24 | 10.10.40.1 |
| 50 | Servers | 10.10.50.0/24 | 10.10.50.1 |
| 99 | Management | 10.10.99.0/24 | 10.10.99.1 |
| 999 | Unused native VLAN | None | None |

For each routed headquarters VLAN, DSW1 uses .2 and DSW2 uses .3. Clients use the HSRP virtual address .1, so a change of active gateway does not require changes to endpoint addressing. HQ-ASW1 has management address 10.10.99.10; the server test endpoint uses 10.10.50.10.

| Connection or segment | Prefix | Endpoint allocation |
| --- | --- | --- |
| HQ-R1 to ISP-R1 | 10.254.0.0/30 | HQ .1; ISP .2 |
| BR-R1 to ISP-R1 | 10.254.0.4/30 | Branch .5; ISP .6 |
| HQ-R1 to HQ-DSW1 | 10.255.1.0/30 | HQ .1; DSW1 .2 |
| HQ-R1 to HQ-DSW2 | 10.255.1.4/30 | HQ .5; DSW2 .6 |
| Branch LAN | 10.20.10.0/24 | Gateway .1; users from .50 |
| External test LAN | 203.0.113.0/24 | Gateway .1; endpoint .10 |
| Device loopbacks | 10.255.0.X/32 | HQ 1; branch 2; ISP 3; DSWs 11, 12 |

### Allocation decisions

Department /24 networks provide a consistent structure and room for growth. This favors readability over tight address conservation. Addresses .1 to .49 are excluded from dynamic user pools, reserving space for gateways and fixed assignments. Server and management addresses are static.

VLAN 999 is an unused native VLAN with no gateway or clients. Trunks carry an explicit list of VLANs, and dynamic trunk negotiation is disabled. VLAN separation limits broadcast domains; restrictions between routed departments depend on access policy, not the VLAN ID alone.


---

## 3 Routing and network services

### Routing domain

OSPFv2 operates in a single area, area 0, across the headquarters edge, distribution switches, provider simulator and branch router. A single area is sufficient for this topology and keeps adjacency and route troubleshooting straightforward. Router identities are assigned explicitly from loopback addresses. OSPF is defined in RFC 2328 [3].

Interfaces are passive by default. Adjacencies are enabled only on routed infrastructure links; user-facing VLAN interfaces advertise their networks without forming neighbors. The distribution switches each have one routed link to HQ-R1. The expected neighbor counts are three at HQ-R1, two at ISP-R1 and one at each remaining OSPF node.

ISP-R1 originates a default route into the lab. This route leads to the connected external test segment; it does not provide internet access. The provider and enterprise share one routing domain solely to keep the private transit simulation compact.

### Address assignment

HQ-R1 hosts DHCP pools for HR, Finance, IT and guests. Both distribution switches relay client broadcasts to the HQ-R1 loopback at 10.255.0.1. The relay configuration exists on both gateway switches, while the DHCP service remains a single point of failure. BR-R1 provides DHCP locally for its branch LAN.

User pools supply the subnet and default gateway. DNS is outside the baseline, so the pools do not advertise an unimplemented resolver. SERVER-PC is a VPCS connectivity target, not a DNS or web server.

### Address translation

Headquarters and branch edges apply PAT to traffic destined for the external simulator. Translation uses the address of each edge router interface facing ISP-R1. NAT classification excludes private inter-site destinations in 10.0.0.0/8 so that the sites retain their original private source addresses when communicating.

A deny statement in the NAT classification ACL means that matching traffic is not translated; it is not a packet-filtering denial. The guest filtering ACL serves a separate purpose and is applied directly to the guest SVI.

### Operational services

NTP forms a local hierarchy: ISP-R1 supplies the lab reference, HQ-R1 and BR-R1 synchronize to it, and headquarters switches use HQ-R1. This aligns timestamps for troubleshooting but is not an independently verified time source. Buffered logs retain recent device events; CDP and LLDP support neighbor identification.

| Traffic example | Forwarding treatment |
| --- | --- |
| HR to server endpoint | Inter-VLAN routing through an HQ distribution switch |
| Branch to headquarters | Private routed path through BR-R1, ISP-R1 and HQ-R1 |
| HQ user to external endpoint | Default route toward ISP-R1 and PAT at HQ-R1 |


---

## 4 Security and availability

### Access policy

The baseline policy limits guest access while allowing communication among trusted departments. The guest ACL is applied inbound on VLAN 40 at both distribution switches. DHCP requests are permitted before RFC 1918 destination ranges are denied; other traffic is then permitted toward the external simulator.

| Source | Destination or service | Policy |
| --- | --- | --- |
| Guest VLAN | DHCP client requests | Permit |
| Guest VLAN | Private internal networks | Deny |
| Guest VLAN | 203.0.113.10 external endpoint | Permit |
| HR Finance and IT | Internal routed departments and server | Permit |
| Branch LAN | Headquarters internal networks | Permit |

These are stateless Layer 3 rules. They do not inspect application behavior, and the baseline does not isolate hosts within the same VLAN. Finance-specific restrictions require an additional application and access matrix.

### Access layer protection

Endpoint ports use access mode, sticky MAC learning and a maximum of one learned address. Port-security violations use restrict mode. PortFast supports endpoint startup, and BPDU Guard protects those ports from unexpected bridge protocol messages. Unused switch interfaces are shut down. SSH, DHCP snooping and Dynamic ARP Inspection procedures are maintained in the separate hardening guide.

### Redundancy design

| Mechanism | Implementation | Failure addressed |
| --- | --- | --- |
| HSRPv2 | DSW1 priority 110; DSW2 100; preemption enabled | Loss of the active default gateway |
| Interface tracking | Routed uplink loss reduces HSRP priority by 20 | Local gateway uplink failure |
| Rapid PVST+ | DSW1 root priority 4096; DSW2 8192 | Loops and access-path selection |
| LACP | Two peer links form Port-channel1 | Loss of one peer bundle member |

HSRP, spanning tree and LACP solve different problems. Losing DSW1’s routed uplink changes its HSRP priority but does not automatically change the spanning-tree root. Traffic may cross the peer bundle to reach DSW2. A complete DSW1 failure also requires the access switch to use its alternate path.


---

## 5 Functional verification

Functional verification covered routed reachability, guest restrictions, address translation and headquarters failover. The methods below define how the configuration is checked and can be reproduced. The assessment is functional: it does not provide throughput, latency or convergence-time benchmarks.

| Area | Verification method | Acceptance condition |
| --- | --- | --- |
| VLANs and trunks | show vlan brief; show interfaces trunk | Correct access membership and matching trunk parameters |
| Link aggregation | show etherchannel summary | Po1 active with both member links bundled |
| Spanning tree | show spanning-tree vlan 10 | DSW1 root and alternate access path visible |
| Routing | show ip ospf neighbor; show ip route | Expected adjacencies and reachable remote prefixes |
| Gateway redundancy | show standby brief | One active gateway and one standby per VLAN |
| DHCP | VPCS lease request; show ip dhcp binding | Correct client subnet and virtual default gateway |
| Internal connectivity | HR and branch ping 10.10.50.10 | Successful reachability to the server test endpoint |
| Guest restriction | Guest ping to 10.10.50.10; ACL counters | Internal traffic denied and matching counter incremented |
| External access | Guest ping 203.0.113.10; NAT table | External reachability with a corresponding PAT entry |
| Configuration persistence | Save configuration and restart device | Saved settings and protocol state recover |

### Failure scenarios

The resilience checks isolate one change at a time. Disabling one peer EtherChannel member tests link aggregation. Disabling DSW1’s routed uplink tests HSRP tracking. Stopping DSW1 tests both gateway takeover and the alternate access-switch path. Each scenario is followed by restoration of the original topology and a repeat connectivity check.

A successful ping alone is not sufficient evidence of the intended mechanism. The forwarding result must be paired with the relevant state: HSRP role, spanning-tree port role, EtherChannel membership, ACL counter or NAT translation. This distinguishes a working control from traffic that succeeds through an unintended path.

### Assessment boundaries

The project validates interactions among network features under a controlled virtual workload. Recovery time can vary with host CPU scheduling and memory pressure. No availability percentage, application performance improvement or production service level is inferred from these checks.


---

## 6 Limitations and conclusion

### Design limitations

Gateway and link redundancy are concentrated at headquarters. HQ-R1, BR-R1, ISP-R1 and HQ-ASW1 remain single points of failure. The headquarters DHCP server is also unpaired. A production design would need an availability target and a failure analysis before deciding where to add capacity and independent paths.

The external network is simulated, and inter-site traffic is not encrypted. The baseline does not include a stateful firewall, application servers, centralized AAA, a SIEM or wireless infrastructure. Physical cabling quality, radio coverage, PoE and hardware forwarding rates cannot be assessed with this topology.

### Further development

The next useful extension is a small Linux service host for DNS, HTTP and centralized logs. That would connect network behavior to application symptoms and create better incident evidence. Firewall and VPN work should follow a defined access policy and a revised memory budget. IPv6, floating static routing and router-on-a-stick remain separate exercises in the supporting materials.

### Conclusion

The project combines department segmentation, dynamic routing, address services and controlled external access in a reproducible enterprise lab. Its main engineering lesson is that resilience depends on coordinated behavior across layers: an available gateway is useful only when a valid Layer 2 path and a working routed uplink also exist. The report and configuration set provide a basis for repeating the checks and extending the design.

### References

[1] Rekhter et al. Address Allocation for Private Internets. RFC 1918, February 1996.
https://www.rfc-editor.org/info/rfc1918/

[2] Arkko et al. IPv4 Address Blocks Reserved for Documentation. RFC 5737, January 2010.
https://www.rfc-editor.org/info/rfc5737/

[3] Moy. OSPF Version 2. RFC 2328, April 1998.
https://www.rfc-editor.org/info/rfc2328/

[4] Cisco. IOSvL2 reference. Cisco Modeling Labs documentation.
https://developer.cisco.com/docs/modeling-labs/iosvl2/

References accessed 8 October 2026. Device-specific behavior depends on the installed image and release. The configuration set and interface schedule are the project-specific implementation record.


---

## Appendix A Interface schedule

Each row represents one virtual Ethernet connection. The two links between the distribution switches are separate LACP members. Other switch uplinks are individual trunks controlled by spanning tree.

| Device | Port | Connected device | Port |
| --- | --- | --- | --- |
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

### Device image record

Routers: vios-adventerprisek9-m-15.6.2T
Switches: viosl2-adventerprisek9-m.ssa.high_iron_20200929
Endpoints: VPCS

The topology archive references these image names and contains no vendor image binaries. The configuration files are organized by device name. IOSvL2 interface numbering advances from Gi0/0–Gi0/3 to Gi1/0–Gi1/3; router ports use Gi0/0–Gi0/3.

Supporting files are organized into configs, docs, evidence and tickets directories. IMPORT_Enterprise_Network_CCNA.zip contains the topology for import into EVE-NG.
