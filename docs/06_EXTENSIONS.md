# Separate exercises after the baseline passes

Save a baseline checkpoint first. These are optional changes, not already applied.

## A. IPv6 routed transit (no extra nodes)

This enables IPv6 on router transit links and loopbacks only. It does not enable guest IPv6 or bypass the IPv4 guest policy. 2001:db8::/32 is documentation space for this isolated lab.

HQ-R1:
```text
configure terminal
ipv6 unicast-routing
interface GigabitEthernet0/0
 ipv6 address 2001:db8:100::1/64
exit
interface Loopback0
 ipv6 address 2001:db8:10::1/128
exit
ipv6 route ::/0 2001:db8:100::2
end
```
ISP-R1:
```text
configure terminal
ipv6 unicast-routing
interface GigabitEthernet0/0
 ipv6 address 2001:db8:100::2/64
exit
interface GigabitEthernet0/1
 ipv6 address 2001:db8:200::2/64
exit
ipv6 route 2001:db8:10::1/128 2001:db8:100::1
ipv6 route 2001:db8:20::1/128 2001:db8:200::1
end
```
BR-R1:
```text
configure terminal
ipv6 unicast-routing
interface GigabitEthernet0/0
 ipv6 address 2001:db8:200::1/64
exit
interface Loopback0
 ipv6 address 2001:db8:20::1/128
exit
ipv6 route ::/0 2001:db8:200::2
end
```
Verify `show ipv6 interface brief`, `show ipv6 neighbors`, `show ipv6 route`. On HQ use `ping ipv6 2001:db8:20::1 source Loopback0`. If the short ping syntax differs, use interactive extended ping. Capture IPv6 neighbor discovery on a transit link. Endpoint SLAAC is a later exercise with a capable Linux host; do not claim it was tested here.

## B. Floating static backup WAN (one new link)

Stop HQ-R1 and BR-R1 before changing their EVE cabling. Add HQ-R1 Gi0/3 to BR-R1 Gi0/2, restart and check saved configs.

HQ-R1:
```text
configure terminal
interface GigabitEthernet0/3
 description PRIVATE_BACKUP_TO_BRANCH
 ip address 10.254.1.1 255.255.255.252
 no shutdown
exit
ip route 10.20.10.0 255.255.255.0 10.254.1.2 200
router ospf 10
 redistribute static subnets
exit
end
```
BR-R1:
```text
configure terminal
interface GigabitEthernet0/2
 description PRIVATE_BACKUP_TO_HQ
 ip address 10.254.1.2 255.255.255.252
 no shutdown
exit
ip route 10.10.10.0 255.255.255.0 10.254.1.1 200
ip route 10.10.20.0 255.255.255.0 10.254.1.1 200
ip route 10.10.30.0 255.255.255.0 10.254.1.1 200
ip route 10.10.40.0 255.255.255.0 10.254.1.1 200
ip route 10.10.50.0 255.255.255.0 10.254.1.1 200
ip route 10.10.99.0 255.255.255.0 10.254.1.1 200
end
```
Do not add this backup link to OSPF. To keep the exercise deterministic, shut BOTH HQ-R1 and BR-R1 Gi0/0 interfaces. Both edges then lose the provider path and install their floating static routes. HQ redistributes its installed branch static route into OSPF so its downstream distribution switches can reach the branch. While the original OSPF route is active, the floating static is not installed and is not redistributed.

Test branch-to-HQ traffic only during this paired outage. External connectivity is intentionally unavailable. A production solution for arbitrary single-link provider failures needs more careful route tracking and redistribution design; this isolated exercise demonstrates administrative distance and backup paths without claiming full WAN resilience.

Verify `show ip route 10.20.10.0` at HQ and `show ip route 10.10.50.0` at branch. Expected static [200/0] replaces OSPF [110/...]. Test BR-PC to server, then restore the failed interfaces and confirm OSPF routes win. Remove the exercise config or keep a separate checkpoint. No NAT should be applied on this private backup link.

## C. Router-on-a-stick (small separate lab)

Stop the main lab to release RAM. Create a separate lab with one fresh IOSv router, one fresh IOSvL2 switch and two VPCS. Use the same image names. Connect router Gi0/0 to switch Gi0/0, PC-A to switch Gi0/1, PC-B to Gi0/2. These commands are for fresh nodes only.

Router:
```text
enable
configure terminal
hostname ROAS-R1
interface GigabitEthernet0/0
 no ip address
 no shutdown
exit
interface GigabitEthernet0/0.110
 encapsulation dot1Q 110
 ip address 10.20.10.1 255.255.255.0
exit
interface GigabitEthernet0/0.120
 encapsulation dot1Q 120
 ip address 10.20.20.1 255.255.255.0
exit
end
write memory
```
Switch:
```text
enable
configure terminal
hostname ROAS-SW1
vlan 110
 name EMPLOYEES
exit
vlan 120
 name SUPPORT
exit
interface GigabitEthernet0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 110,120
 no shutdown
exit
interface GigabitEthernet0/1
 switchport mode access
 switchport access vlan 110
 spanning-tree portfast
 no shutdown
exit
interface GigabitEthernet0/2
 switchport mode access
 switchport access vlan 120
 spanning-tree portfast
 no shutdown
exit
end
write memory
```
PC-A: `ip 10.20.10.10/24 10.20.10.1`; PC-B: `ip 10.20.20.10/24 10.20.20.1`. Run `save` on both, then ping between PCs. Capture the router trunk to inspect 802.1Q tags.
