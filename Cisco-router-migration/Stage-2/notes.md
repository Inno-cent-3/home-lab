---

# Stage 2 — NAT, DHCP, and ACLs (Clean Rebuild)

This stage enhances the Cisco router by refining DHCP, preparing NAT for future services, and adding basic LAN security ACLs.

---

## Goals
- Prepare DHCP for Pi-hole integration
- Maintain clean NAT configuration
- Add LAN security ACLs to block private/bogon ranges
- Document everything for GitHub

---

## DHCP Refinement
DHCP pool configured for the lab:

- Subnet: 192.168.50.0/24
- Gateway: 192.168.50.1
- DNS (temporary): 8.8.8.8  
  *(will be replaced with Pi-hole in Stage 4)*

---

## NAT Configuration
NAT overload allows LAN devices to access the internet:

```
ip nat inside source list 1 interface GigabitEthernet0/0 overload
```

This will later support:
- Pi-hole
- WireGuard VPN
- RDP server
- Additional lab services

---

## LAN Security ACL
A basic ACL was added to block RFC1918 inbound traffic and bogon ranges:

```
ip access-list extended LAN-SECURITY
 permit ip 192.168.50.0 0.0.0.255 any
 deny ip 10.0.0.0 0.255.255.255 any
 deny ip 172.16.0.0 0.15.255.255 any
 deny ip 192.168.0.0 0.0.255.255 any
 deny ip 0.0.0.0 0.255.255.255 any
 deny ip 100.64.0.0 0.63.255.255 any
 deny ip 169.254.0.0 0.0.255.255 any
 permit ip any any
```

Applied to LAN interface:

```
interface GigabitEthernet0/1
 ip access-group LAN-SECURITY in
```

---

## Tests
- LAN devices receive DHCP correctly  
- Internet access works  
- ACL blocks unwanted inbound private ranges  
- NAT continues functioning normally  

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