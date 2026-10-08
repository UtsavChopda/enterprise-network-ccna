# Acceptance tests — record actual results, not assumed passes

Run `show ip interface brief` on all network devices first. Save outputs/screenshots with timestamps. Boot and convergence time depend on your CPU and images.

| Test | Commands / action | Expected result |
|---|---|---|
| VLANs / trunks | `show vlan brief`, `show interfaces trunk` | Correct access VLANs, matching native 999, allowed VLANs |
| LACP | `show etherchannel summary` | Po1 up, both members bundled (P) |
| STP | `show spanning-tree vlan 10` | DSW1 root; access switch has an alternate path |
| OSPF | `show ip ospf neighbor`, `show ip route ospf` | HQ has 3 neighbors, ISP 2, Branch 1, each DSW 1 |
| HSRP | `show standby brief` on both DSWs | DSW1 active, DSW2 standby, virtual .1 |
| DHCP | `ip dhcp` then `show ip` on VPCS | Correct subnet and .1 gateway |
| DHCP leases | `show ip dhcp binding` on HQ and branch | Leases recorded |
| Inter-VLAN | HR `ping 10.10.50.10` | Server placeholder responds |
| Branch | BR-PC `ping 10.10.50.10` | Private routed connection succeeds |
| Guest denial | GUEST-PC `ping 10.10.50.10` | Fails; GUEST_IN deny counter increments |
| Guest external | GUEST-PC `ping 203.0.113.10` | Succeeds |
| PAT | Repeat external ping; `show ip nat translations` on HQ | Translation visible while active (entries expire) |
| Time | `show ntp associations`, `show clock` | Synchronization eventually follows lab ISP clock |
| Link failure | Continuous HR ping to server; shut DSW1 Gi0/1 | Po1 stays up on second member |
| Gateway uplink loss | Shut DSW1 Gi0/0 | Its HSRP priority falls to 90; DSW2 becomes active |
| Gateway node failure | Stop DSW1; ping server from HR | ASW alternate path and DSW2 restore service; record loss |
| Restore | `no shutdown` affected ports / restart node | Baseline state returns |

Do one failure at a time. Never save intentional broken configurations. Single HQ edge router, branch edge, ISP, and access switch remain single points of failure; gateway redundancy is not end-to-end high availability. DHCP on HQ-R1 is not redundant.

Fault tickets to create after a working baseline:
1. Remove VLAN 20 from ASW's active trunk. Investigate trunk/STP state; note the alternate trunk is a separate path, so evaluate actual reachability. Restore allowed VLAN list.
2. Set BR-R1 Gi0/0 OSPF area to 1 using interface configuration. Check neighbor/interface area, remove the override to restore area 0.
3. Remove BOTH guest SVI relay statements, release/renew the guest lease; restore both helpers to 10.255.0.1. Existing leases do not immediately disappear.
4. Insert a deny above the relevant permit in a test ACL. Inspect sequence order and counters; remove the test ACE.
5. Change an endpoint subnet mask and explain local ARP versus gateway selection.

Ticket template: time, symptom, expected behavior, commands/output, hypothesis, root cause, fix, verification, rollback, measured recovery or packet loss.

Topology import was confirmed by screenshot, and the project owner subsequently confirmed completion of core configuration and testing. Preserve command output and evidence for independent review.
