# 🌐 Enterprise Network Design & Implementation

![Platform](https://img.shields.io/badge/Platform-EVE--NG-005A9C)
![Networking](https://img.shields.io/badge/Networking-Cisco_IOS-049FD9)
![Focus](https://img.shields.io/badge/Focus-Routing_%26_Switching-2563EB)
![Security](https://img.shields.io/badge/Security-Segmentation_%26_ACLs-15803D)

## 📌 Project Overview

A multi-site enterprise network built in EVE-NG, connecting a headquarters and branch through a simulated provider. The project combines departmental segmentation, dynamic routing, redundant headquarters gateways, and controlled access to an external test network.

**Author:** Utsav Chopda · **Platform:** EVE-NG · **Lab:** 13 nodes / 15 links

[Project report (PDF)](reports/Utsav_Chopda_Network_Project_Report.pdf) · [Read the report](reports/Project_Report.md) · [Import the lab](lab/IMPORT_Enterprise_Network_CCNA.zip)

**Quick navigation:** [Architecture](#architecture) · [Features](#features) · [Setup](#setup) · [Verification](#verification) · [Documentation](#documentation)

## 🎯 Business Scenario & Objectives

A fictional company operates a Bengaluru headquarters and a Pune branch. HR, Finance, IT, guests, and shared resources require separate network segments, while employees need routed connectivity between offices.

The design addresses four requirements:

- Separate departmental broadcast domains and isolate guests from private networks.
- Provide automatic client addressing and dynamic routing between sites.
- Maintain headquarters gateway connectivity during selected switch or uplink failures.
- Document the topology, configuration, and operational checks so the lab can be reproduced.

## 🛠️ Technologies & Tools

| Technology | Role in the project |
| --- | --- |
| EVE-NG | Virtual lab platform |
| Cisco IOSv | Headquarters, branch, and simulated provider routers |
| Cisco IOSvL2 | Distribution and access switching |
| VPCS | Seven lightweight endpoints for addressing and reachability checks |
| Cisco IOS CLI | Device configuration, state inspection, and troubleshooting |
| GitHub & Markdown | Versioned configurations and technical documentation |


<a id="architecture"></a>

## 🗺️ Network Architecture

![Enterprise network topology](Topology.svg)

The headquarters uses two distribution switches and one access switch. A separate router serves the branch. Three IOSv routers, three IOSvL2 switches, and seven VPCS endpoints provide the complete lab environment.

<a id="features"></a>

## 🚀 Key Features

| Area | Implementation |
| --- | --- |
| Segmentation | Departmental VLANs, 802.1Q trunks, SVIs, dedicated management VLAN |
| Switching | Rapid PVST+, LACP EtherChannel, PortFast, BPDU Guard |
| Routing | Single-area OSPFv2, passive interfaces, advertised default route |
| Gateway redundancy | HSRPv2, preemption, routed-uplink tracking |
| Address assignment | Central DHCP at headquarters, DHCP relay, local branch DHCP |
| External access | PAT with inter-site traffic excluded from translation |
| Access controls | Guest IPv4 ACLs, sticky port security, unused-port shutdown |
| Operations | NTP, CDP/LLDP, buffered logging, verification and failure exercises |

## 📋 IP Addressing & VLAN Plan

| Segment | Subnet | Gateway |
| --- | --- | --- |
| HR / VLAN 10 | 10.10.10.0/24 | 10.10.10.1 |
| Finance / VLAN 20 | 10.10.20.0/24 | 10.10.20.1 |
| IT / VLAN 30 | 10.10.30.0/24 | 10.10.30.1 |
| Guest / VLAN 40 | 10.10.40.0/24 | 10.10.40.1 |
| Server / VLAN 50 | 10.10.50.0/24 | 10.10.50.1 |
| Management / VLAN 99 | 10.10.99.0/24 | 10.10.99.1 |
| Branch LAN | 10.20.10.0/24 | 10.20.10.1 |
| External test LAN | 203.0.113.0/24 | 203.0.113.1 |

Headquarters gateways are HSRP virtual addresses. See the [addressing guide](docs/02_ADDRESSING.md) for physical SVI addresses, loopbacks, and routed transit links.

<a id="setup"></a>

## ⚙️ Installation & Setup

### Requirements

- EVE-NG with hardware virtualization available.
- Matching, legally obtained IOSv and IOSvL2 images; image identifiers are documented in the report.
- An 8 GB EVE-NG VM, with sufficient CPU and memory headroom on the host.
- Console access to the virtual devices.

### Get the project

```bash
git clone https://github.com/UtsavChopda/enterprise-network-ccna.git
cd enterprise-network-ccna
```

Alternatively, download the repository through **Code → Download ZIP**.

### Import and configure

1. Provide licensed IOSv and IOSvL2 images in EVE-NG. Image identifiers and interface assignments are listed in the report and [cabling guide](docs/01_CABLING.md).
2. Import `lab/IMPORT_Enterprise_Network_CCNA.zip`. The editable `.unl` topology is also included.
3. Start the network devices sequentially, then the VPCS clients. This project uses an EVE-NG VM with 8 GB RAM; the six network nodes reserve 6 GB before overhead.
4. Open each console and apply the matching file from `configs/`. These are console-paste configurations; the topology archive does not automatically load them.
5. Follow the [verification guide](docs/03_VERIFY_AND_TROUBLESHOOT.md) to check forwarding, isolation, and redundancy. Restore the baseline after each failure exercise.

Image command support and available memory can vary. Review any rejected commands before proceeding.

## 🧠 Design Decisions

- **Gateway and switching roles align.** DSW1 is the preferred HSRP gateway and spanning-tree root; DSW2 provides the secondary path.
- **Uplink tracking affects gateway selection.** A failed routed uplink lowers the active switch's HSRP priority.
- **Guest access is constrained at the gateway.** DHCP remains available while private destination networks are denied.
- **Inter-site traffic preserves source addresses.** The PAT selection rules exclude internal destinations.

<a id="verification"></a>

## 🧪 Verification & Failure Scenarios

The following checks explain how to evaluate the design. They describe acceptance criteria, rather than measured performance results.

| Scenario | Expected behavior | Useful checks |
| --- | --- | --- |
| Departmental addressing | Clients receive addresses from the correct VLAN pool | `show ip dhcp binding`; client `show ip` |
| Site-to-site routing | HQ and branch networks are reachable over learned routes | `show ip ospf neighbor`; `show ip route ospf` |
| Guest isolation | Guest traffic to private networks is denied; external test access is allowed | `show access-lists`; targeted client pings |
| Address translation | External test traffic creates PAT entries; inter-site traffic retains its source address | `show ip nat translations` |
| Gateway uplink failure | HSRP tracking lowers priority and the secondary gateway takes over | `show standby brief`; continuous reachability probes |
| EtherChannel member failure | The bundle continues forwarding over the remaining member | `show etherchannel summary` |
| Layer 2 redundancy | Spanning tree maintains a loop-free forwarding path | `show spanning-tree` |

Follow the [verification and troubleshooting guide](docs/03_VERIFY_AND_TROUBLESHOOT.md) for the exercise sequence. The [test register](evidence/TEST_RESULTS.md) records completion status, and the [incident template](tickets/INCIDENT_TEMPLATE.md) provides a format for recording symptoms, diagnosis, and recovery.

## 💡 Skills Demonstrated

- IPv4 subnet planning, departmental segmentation, and interface mapping.
- Cisco routing and switching configuration across a multi-site topology.
- Gateway redundancy and Layer 2 resilience using HSRP, spanning tree, and LACP.
- DHCP relay, NAT selection rules, and IPv4 access-control policy.
- Structured troubleshooting using forwarding tests and protocol state.
- Technical reporting and reproducible lab documentation.

<a id="documentation"></a>

## 📂 Repository Structure & Documentation

| File or directory | Purpose |
| --- | --- |
| [reports/](reports/) | Formal technical report in PDF, Word, and Markdown |
| [lab/](lab/) | Import archive and editable EVE-NG topology |
| [configs/](configs/) | Device-specific configuration files |
| [evidence/](evidence/) | Test register and evidence references |
| [tickets/](tickets/) | Troubleshooting incident template |
| [Cabling](docs/01_CABLING.md) | Port-to-port interface schedule |
| [Addressing](docs/02_ADDRESSING.md) | VLAN, subnet, and transit plan |
| [Verification](docs/03_VERIFY_AND_TROUBLESHOOT.md) | Acceptance checks and troubleshooting exercises |
| [Security extensions](docs/04_SSH_AND_SECURITY.md) | SSH, DHCP snooping, and DAI exercises |
| [Topic coverage](docs/05_TOPIC_COVERAGE_AND_EXTENSIONS.md) | Core coverage and extension scope |
| [Additional exercises](docs/06_EXTENSIONS.md) | IPv6, floating static routes, and router-on-a-stick |

## 🔎 Project Scope

This is an educational emulation. The provider router participates in the same OSPF domain, and the external endpoint simulates outside connectivity. SERVER-PC is a VPCS reachability target. The project does not provide real internet access, application hosting, a stateful firewall, or an encrypted WAN.

Gateway redundancy does not eliminate the edge-router, access-switch, or DHCP single points of failure. Optional exercises are documented separately from the core design. The repository provides verification procedures; no measured throughput or failover-time claims are made.

Vendor images, credentials, and private keys are excluded.

## 👤 Author

**Utsav Chopda**

Networking and cybersecurity portfolio project focused on enterprise routing, switching, and access control.

[GitHub profile](https://github.com/UtsavChopda)

## 🔗 Related Projects

| Repository | Focus |
| --- | --- |
| [Network Scanner](https://github.com/UtsavChopda/network-scanner) | Local network discovery using ARP |
| [SIEM Log Analyzer](https://github.com/UtsavChopda/siem-log-analyzer) | Authentication-log analysis and brute-force detection |
| [Linux Knowledge Base](https://github.com/UtsavChopda/Linux-Knowledge-Base) | Linux administration, networking, and security learning |
