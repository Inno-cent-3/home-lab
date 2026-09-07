
---

# Stage 4 — DNS Enforcement + Pi‑hole Integration (Clean Rebuild)

This stage completes the DNS hardening of the homelab by integrating Pi‑hole with the Cisco router, fixing DNS leaks on the Ubuntu VM, enforcing DNS for all LAN clients, and validating the full DNS chain from client → Pi‑hole → Quad9.

---

## Goals
- Enforce Pi‑hole as the only DNS resolver for the LAN  
- Update Cisco DHCP to point DNS → 192.168.50.10  
- Fix Ubuntu netplan, systemd‑resolved, and resolv.conf  
- Remove DNS leaks (8.8.8.8, fallback resolvers, DHCP overrides)  
- Validate DNS chain using dig and Pi‑hole logs  
- Document everything for GitHub  

---

## Cisco Router Configuration (DNS Enforcement)

### DHCP DNS Update  
The router now forces all LAN clients to use Pi‑hole:

```
ip dhcp pool MY-NETWORK
 default-router 192.168.50.1
 dns-server 192.168.50.10
```

This replaces Google DNS (8.8.8.8) from Stage 1.

### ACL Updates  
DHCP and DNS traffic must be allowed:

```
ip access-list extended LAN-SECURITY
 remark Allow DHCP
 permit udp any any eq bootps
 permit udp any any eq bootpc

 remark Allow DNS
 permit udp any any eq domain
 permit tcp any any eq domain

 remark Allow LAN
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

Applied to LAN:

```
interface GigabitEthernet0/1
 ip access-group LAN-SECURITY in
```

### NAT Validation  
NAT continues to work normally:

```
ip nat inside source list 1 interface GigabitEthernet0/0 overload
```

---

## Ubuntu VM Fixes (DNS Leak Removal)

### Netplan Fix  
The VM was leaking DNS to 8.8.8.8 due to a leftover netplan entry.  
Final working configuration:

```
nameservers:
  addresses:
    - 127.0.0.1
```

Applied with:

```
sudo netplan apply
```

### systemd‑resolved Fix  
Ensures Pi‑hole is used locally:

```
DNS=127.0.0.1
FallbackDNS=9.9.9.9
DNSStubListener=yes
```

### resolv.conf Fix  
Final working output:

```
nameserver 127.0.0.1
nameserver 127.0.0.1
nameserver 192.168.50.10
```

This confirms no fallback to Google DNS.

---

## Pi‑hole Configuration

### Upstream DNS  
Quad9 selected for privacy and security:

```
9.9.9.9
149.112.112.112
```

### Listening Behavior  
Pi‑hole listens on:

- 127.0.0.1#53  
- 192.168.50.10#53  

### Validation  
Pi‑hole logs show:

- LAN clients → Pi‑hole  
- Pi‑hole → Quad9  
- No leaks  
- No bypass  
- No fallback resolvers  

---

## DNS Chain Verification

### dig Test  
From the Ubuntu VM:

```
dig google.com
```

Expected and confirmed:

- SERVER: 127.0.0.1#53  
- Answer from Quad9  
- Query time < 20 ms  
- No 8.8.8.8  
- No errors  

### LAN Client Test  
From any LAN device:

```
nslookup google.com
```

Expected:

- Server: 192.168.50.10  
- Answer from Quad9  

---

## Stage‑4 Topology Diagram

A diagram was generated showing:

- Bell Modem → Cisco Router → Pi‑hole → Quad9  
- Cisco Router → LAN Client → Pi‑hole  
- DHCP DNS enforcement  
- ACL filtering  
- NAT inside/outside  
- Pi‑hole upstream DNS  

Saved as:

```
dns-flow-diagram.png
```

---

## Lessons Learned
- Netplan overrides everything — fix it first  
- DHCP DNS must point to Pi‑hole for full enforcement  
- ACLs must explicitly allow DHCP + DNS  
- resolv.conf must be corrected to avoid leaks  
- Pi‑hole + Quad9 provides secure, filtered DNS  
- DNS enforcement is the foundation for Stage‑5 WireGuard  

---

## Files Included in This Stage
- `show-run.txt`  
- `show-access-lists.txt`  
- `show-ip-int-brief.txt`  
- `show-ip-route.txt`      
- `ubuntu-netplan.yaml`  
- `resolved.conf`  
- `resolv.conf-output.txt`  
- `dig-test.txt`    
- `dns-flow-diagram.png`  
- `notes.md`  

---

Stage 4 complete.  
Next step: **Stage 5 — WireGuard VPN + Remote DNS Enforcement**  