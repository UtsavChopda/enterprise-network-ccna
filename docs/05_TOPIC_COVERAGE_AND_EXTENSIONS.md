# Coverage — configured versus later work

The supplied core is broad, but it is not a claim that every CCNA objective has already been implemented or tested.

| Status | Topics |
|---|---|
| Core configuration and testing completed per project owner | IPv4 / VLSM, VLANs, 802.1Q/native VLAN, SVIs, Rapid PVST+, root priorities, PortFast/BPDU Guard, LACP, CDP/LLDP, single-area OSPF, passive interfaces, loopbacks, default-route advertisement, DHCP/relay, PAT with private-traffic exemption, extended ACL, port security, HSRPv2/link tracking, NTP, buffered logs |
| Manual follow-on provided | SSH/local authentication/VTY ACL; DHCP snooping and DAI with lease-binding prerequisite |
| Ready-to-apply separate exercise | IPv6 router transit + IPv6 static/default routing; floating static route failover; router-on-a-stick branch |
| Requires Linux/service setup | DNS, HTTP/HTTPS, actual TCP/UDP captures, remote syslog, SNMP, automation script execution |
| Requires additional images/resources or design | Stateful firewall/DMZ, site-to-site VPN, central AAA, Splunk, WLAN controller |
| Explain / demonstrate separately | Wireless RF/channels/WPA modes, PoE, copper/fiber faults, QoS behavior, REST/JSON/controllers, cloud architectures |

Use `docs/06_EXTENSIONS.md` for exact isolated exercises. Complete baseline tests before altering it. Keep separate saved exports for each exercise so baseline routes and security behavior are reproducible.

Your Tiny Core image is linux-tinycore-6.4 and your Windows image is win-7-x86. Both were configured for 4096 MB in the uploaded file. Neither is included in the initial topology. A VPCS named SERVER-PC is only an ICMP endpoint, not a web/DNS server. Linux can replace it later when memory is available and required software has been checked. Keep the legacy Windows image out of this first build.

Portfolio: business requirements, design rationale, addressing/cabling, sanitized configs, actual acceptance results, 5 troubleshooting tickets, packet captures and 3-minute demo. Describe this as a virtual lab, not a production deployment. Only write completion claims after tests pass.

Reference sources consulted:
- https://www.eve-ng.net/index.php/how-to-eve-ng-api/
- https://developer.cisco.com/docs/modeling-labs/iosvl2/
- User's supplied PROJECT-1.unl (authoritative image names, templates, RAM, QEMU options).
Cisco's current IOSvL2 documentation describes HSRP/SVI and switching capabilities; your older exact image still needs runtime verification.
