# Stage 1 — Cisco Router Migration (Clean Rebuild)

This stage recreates the initial setup of my homelab network using a Cisco enterprise router.  
The goal is to build a clean WAN/LAN foundation using safe IP ranges for public documentation.

---

## Goals
- Establish a clean WAN/LAN architecture
- Use safe, non‑production IP ranges (192.168.50.0/24)
- Configure NAT inside/outside
- Enable DHCP for the lab network
- Prepare the environment for future stages (Pi‑hole, WireGuard, ACLs)
- Document everything for GitHub

---

## Network Topology
```
Bell Modem / ISP
        |
        |  (WAN – DHCP)
        v
Cisco Router (R1)
G0/0: WAN (DHCP)
G0/1: LAN 192.168.50.1/24
NAT: inside/outside
        |
        |  (LAN – 192.168.50.0/24)
        v
Ubuntu VM (Lab Server)
192.168.50.x
```

---

## WAN Configuration
- WAN interface receives IP via DHCP from ISP  
- NAT outside enabled  
- Default route tracks DHCP gateway  

---

## LAN Configuration
- LAN subnet: **192.168.50.0/24**
- Router LAN IP: **192.168.50.1**
- NAT inside enabled
- DHCP pool configured:
  - Default gateway: 192.168.50.1
  - DNS (temporary): 8.8.8.8  
    *(will be replaced with Pi‑hole in Stage 4)*

---

## NAT Configuration
```
ip nat inside source list 1 interface GigabitEthernet0/0 overload
access-list 1 permit 192.168.50.0 0.0.0.255
```

This allows all LAN devices to access the internet through the WAN interface.

---

## Connectivity Tests
Performed from the Ubuntu VM:

- Ping router LAN: `ping 192.168.50.1`  
- Ping internet: `ping 8.8.8.8`  
- DNS test: `ping google.com`  
  *(DNS will be fully configured in Stage 4)*

All tests successful.

---

## Lessons Learned
- Cisco routers require explicit NAT inside/outside configuration
- DHCP exclusions prevent conflicts with static devices
- Keeping WAN/LAN simple makes future ACL and DNS enforcement easier
- Using safe IP ranges allows public documentation without exposing the real homelab

---

## Files Included in This Stage
- `show-run.txt`
- `show-ip-int-brief.txt`
- `show-ip-route.txt`
- `topology.png`
- `notes.md`

---

Stage 1 complete.  
Next step: **Stage 2 — NAT + DHCP + ACLs**
```