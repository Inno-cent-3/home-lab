
---

# Stage 2 — NAT, DHCP, and ACLs (Clean Rebuild)

This stage enhances the Cisco router by refining DHCP, preparing NAT for future services, and adding LAN security ACLs. A critical fix was applied to ensure DHCP traffic is not blocked by the ACL.

---

## Goals
- Prepare DHCP for Pi-hole integration
- Maintain clean NAT configuration
- Add LAN security ACLs to block private/bogon ranges
- Fix ACL to allow DHCP traffic
- Document everything for GitHub

---

## DHCP Configuration
DHCP pool for the lab:

```
ip dhcp excluded-address 192.168.50.1 192.168.50.10
ip dhcp pool MY-NETWORK
 network 192.168.50.0 255.255.255.0
 default-router 192.168.50.1
 dns-server 8.8.8.8
```

This will later be updated to use Pi-hole (192.168.50.10) in Stage 4.

---

## NAT Configuration
```
ip nat inside source list 1 interface GigabitEthernet0/0 overload
access-list 1 permit 192.168.50.0 0.0.0.255
```

This provides internet access for all LAN devices.

---

## LAN Security ACL (Updated)
A DHCP fix was required because DHCP clients use source IP **0.0.0.0**, which was previously blocked.

Updated ACL:

```
ip access-list extended LAN-SECURITY
 remark Allow DHCP traffic
 permit udp any any eq bootps
 permit udp any any eq bootpc

 remark Allow LAN to anywhere
 permit ip 192.168.50.0 0.0.0.255 any

 remark Block RFC1918 inbound
 deny ip 10.0.0.0 0.255.255.255 any
 deny ip 172.16.0.0 0.15.255.255 any
 deny ip 192.168.0.0 0.0.255.255 any

 remark Block bogon ranges
 deny ip 0.0.0.0 0.255.255.255 any
 deny ip 100.64.0.0 0.63.255.255 any
 deny ip 169.254.0.0 0.0.255.255 any

 remark Allow everything else
 permit ip any any
```

Applied to LAN interface:

```
interface GigabitEthernet0/1
 ip access-group LAN-SECURITY in
```

---

## DHCP Fix Summary
Without the DHCP permit rules, devices could not obtain IP addresses because the ACL blocked packets from **0.0.0.0 → 255.255.255.255**.

Adding:

```
permit udp any any eq bootps
permit udp any any eq bootpc
```

resolved the issue.

---

## Tests
- Devices now receive IP addresses correctly
- NAT continues to work
- ACL blocks unwanted inbound private ranges
- DHCP traffic flows normally

---

## Files Included in This Stage
- `show-run.txt`
- `show-access-lists.txt`
- `show-ip-int-brief.txt`
- `show-ip-route.txt`
- `notes.md`

---

Stage 2 complete.  
Next step: **Stage 3 — Fresh Ubuntu VM + Pi-hole Installation**
```

---