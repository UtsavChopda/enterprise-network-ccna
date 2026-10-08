# Add SSH after baseline connectivity

Configs contain no reused EVE credentials. From each Cisco console, replace `<YOUR_NEW_LAB_SECRET>` locally before running:

```text
configure terminal
username netadmin privilege 15 secret <YOUR_NEW_LAB_SECRET>
crypto key generate rsa modulus 2048
ip ssh version 2
ip access-list standard MGMT_SSH
 permit 10.10.30.0 0.0.0.255
 deny any log
exit
line vty 0 4
 login local
 transport input ssh
 access-class MGMT_SSH in
 exec-timeout 10 0
exit
end
write memory
```

If a model exposes additional VTY lines, apply the same settings to them. An IT Linux host is needed to test SSH; VPCS cannot run an SSH client. Keep console access while testing. Save only sanitized examples to GitHub, never credentials or private keys. These are device CLI credentials, not EVE web credentials.

Optional DHCP snooping/DAI stage on HQ-ASW1 only, after DHCP success:

```text
configure terminal
ip dhcp snooping
ip dhcp snooping vlan 10,20,30,40
no ip dhcp snooping information option
interface range GigabitEthernet0/0 - 1
 ip dhcp snooping trust
 ip arp inspection trust
exit
end
```

Renew ALL four departmental VPCS leases. Confirm populated `show ip dhcp snooping binding` BEFORE enabling `ip arp inspection vlan 10,20,30,40` in configuration mode. Static server and management VLANs are deliberately excluded. Inspect `show ip arp inspection statistics`. Commands depend on image support. To roll back, disable inspection for those VLANs first, then snooping. Do not trust endpoint ports. Port security in baseline uses sticky learning and restrict mode; replacing VPCS with Linux may require clearing the previously learned sticky MAC on that access port.
