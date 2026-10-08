# Addressing and policy

| Segment | Subnet | Addresses |
|---|---|---|
| HQ–ISP | 10.254.0.0/30 | HQ .1, ISP .2 |
| Branch–ISP | 10.254.0.4/30 | Branch .5, ISP .6 |
| HQ–DSW1 | 10.255.1.0/30 | HQ .1, DSW1 .2 |
| HQ–DSW2 | 10.255.1.4/30 | HQ .5, DSW2 .6 |
| HR / Finance / IT / Guest / Server / Management | 10.10.VLAN.0/24 | HSRP .1, DSW1 .2, DSW2 .3 |
| Branch | 10.20.10.0/24 | Router .1 |
| External simulation | 203.0.113.0/24 | ISP .1, client .10 |
| Loopbacks | 10.255.0.X/32 | HQ 1, Branch 2, ISP 3, DSW1 11, DSW2 12 |

HQ DHCP leases begin at .50 in VLANs 10,20,30,40. Servers and management use static addresses. DHCP supplies no DNS server until a real lab resolver is deployed. VLAN 999 is an unused native VLAN, with no SVI or clients. /24 department networks reserve growth capacity; point-to-point /30s demonstrate VLSM. A smaller real deployment could use smaller department subnets.

Baseline policy: non-guest users may communicate across departments and branches. Guests may reach the external simulator, but not RFC1918 internal destinations (DHCP is permitted first). Finance-specific restrictions are a separate change exercise, not implemented by baseline configs. ACLs here are stateless and do not constitute a stateful firewall.

ISP-R1 is a combined simulated private WAN transit and external-test provider. It participates in the lab's OSPF domain for teaching; this is not a model of OSPF peering with a public internet ISP. Its synthetic OSPF default leads only to the connected external test subnet. There is no real internet, public DNS, VPN encryption, or external cloud bridge. Private site traffic is excluded from PAT. External traffic is translated separately at HQ and branch.
