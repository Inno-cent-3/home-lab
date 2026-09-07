
---

# Stage 3 — Fresh Ubuntu VM + Pi-hole Installation (Clean Rebuild)

This stage rebuilds the DNS infrastructure of the homelab using a fresh Ubuntu VM and a clean Pi-hole installation. The VM will also host WireGuard in Stage 5, making it the central service node of the lab.

---

## Goals
- Deploy a fresh Ubuntu VM for DNS and future VPN services
- Assign a static IP (192.168.50.10)
- Install Pi-hole cleanly with safe blocklists
- Fix hostname resolution issues
- Prepare for DNS enforcement in Stage 4
- Document everything for GitHub

---

## Ubuntu VM Setup
- Updated system packages using `apt update && apt upgrade`
- Set hostname: `pihole-lab`
- Fixed `/etc/hosts` mismatch:
  ```
  127.0.1.1   pihole-lab
  ```
- Assigned static IP: `192.168.50.10`
- Verified connectivity:
  - Router: `ping 192.168.50.1`
  - Internet: `ping 8.8.8.8`
  - DNS: `ping google.com`

---

## Pi-hole Installation
Installed using:

```
curl -sSL [https://install.pi-hole.net](https://install.pi-hole.net) | bash
```

Configuration:
- Interface: ens33
- Static IP: 192.168.50.10
- Upstream DNS: Google (temporary)
- DNSSEC enabled
- Web admin enabled
- Default blocklists only

Pi-hole admin panel:
- http://192.168.50.10/admin

---

## Pi-hole Debug Information
Pi-hole v6+ stores debug logs at:

```
/var/log/pihole/pihole_debug.log
```

A debug token was generated for verification:
- https://tricorder.pi-hole.net/soGFcMaM/

Files added to GitHub:
- `pihole-version.txt`
- `pihole-debug.txt`

---

## Cisco DHCP Update
DHCP pool updated to use Pi-hole:

```
dns-server 192.168.50.10
```

All LAN clients now use Pi-hole for DNS.

---

## Tests
- Pi-hole admin loads correctly
- DNS queries appear in Pi-hole logs
- LAN devices resolve domains through Pi-hole
- Ads blocked successfully
- Hostname resolution fixed (sudo works normally)

---

## Files Included in This Stage
- `pihole-version.txt`
- `pihole-debug.txt`
- `topology.png`
- `notes.md`

---

Stage 3 complete.  
Next step: **Stage 4 — DNS Enforcement + Firewall Rules**
```

---