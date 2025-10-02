# Homelab Master ID & Device Inventory (Updated)

## Proxmox Hosts
| ID  | Name             | FQDN             | Role | VLAN | IP          | Notes |
|-----|------------------|------------------|------|------|-------------|-------|
| 000 | proxmox          | proxmox.home.lab | Main hypervisor | 10 | 10.20.10.2 | Old gaming PC (currently running VMs) |
| 001 | dell-proxmox     | dell.home.lab    | Future hypervisor | 10 | 10.20.10.3 | Dell server, will become main Proxmox host |

---

## Core Infrastructure
| VMID | Name     | FQDN            | Type | VLAN | IP          | Memory | Notes |
|------|----------|-----------------|------|------|-------------|--------|-------|
| 100  | OPNsense | router.home.lab | VM   | —    | WAN: DHCP   | 3 GB   | Firewall/router, runs inside Proxmox but acts as independent machine |
|      |          |                 |      | 10   | 10.20.10.1  |        | Default LAN gateway |
| 300  | NAS      | nas.home.lab    | VM   | 10   | 10.20.10.10 | 4 GB+  | File storage, backups |
| 350  | PBS      | pbs.home.lab    | VM   | 10   | 10.20.10.60 | TBD    | Proxmox Backup Server (future) |

---

## Network Services
| VMID | Name          | FQDN           | Type | VLAN | IP           | Memory | Notes |
|------|---------------|----------------|------|------|--------------|--------|-------|
| 210  | Pi-hole       | dns.home.lab   | LXC  | 10   | 10.20.10.53  | 0.5 GB | DNS/ad-block |
| 220  | Home Assistant| ha.home.lab    | VM   | 20   | 10.20.20.5   | 3 GB   | IoT automation (HAOS VM) |

---

## Application Stack (Docker)
| VMID | Name   | FQDN           | Type | VLAN | IP           | Memory | Notes |
|------|--------|----------------|------|------|--------------|--------|-------|
| 200  | Docker | apps.home.lab  | VM   | 10   | 10.20.10.20  | 4–6 GB | Runs Docker & Compose |

### Docker AppIDs
| AppID | Name         | FQDN                            | VLAN | IP (reverse proxy) | Notes |
|-------|--------------|---------------------------------|------|--------------------|-------|
| 210   | Heimdall     | heimdall.container.home.lab     | 10   | 10.20.10.20        | Dashboard |
| 220   | Linkding     | linkding.container.home.lab     | 10   | 10.20.10.20        | Bookmarks |
| 230   | Homebox      | homebox.container.home.lab      | 10   | 10.20.10.20        | Inventory |
| 240   | Uptime Kuma  | uptime.container.home.lab       | 10   | 10.20.10.20        | Monitoring |
| 250   | Portainer    | portainer.container.home.lab    | 10   | 10.20.10.20        | Docker mgmt |
| 260   | LanguageTool | languagetool.container.home.lab | 10   | 10.20.10.20        | Grammar tool |
| 270   | Traefik      | traefik.container.home.lab      | 10   | 10.20.10.20        | Reverse proxy |

---

## Clients & Devices (LAN — VLAN 10)
| ID  | Device           | FQDN              | VLAN | IP           | Notes |
|-----|------------------|-------------------|------|--------------|-------|
| 400 | Gaming PC        | ellie-pc.home.lab | 10   | 10.20.10.100 | Static IP |
| 401 | PlayStation 5    | ps5.home.lab      | 10   | 10.20.10.151 | Wired |
| 402 | Nintendo Switch  | switch.home.lab   | 10   | 10.20.10.152 | Wired |
| 403 | Xbox Series X    | xbox.home.lab     | 10   | 10.20.10.153 | Wired |
| 404 | 3D Printer #1    | printer3d.home.lab| 10   | Wi-Fi (DHCP) | Wi-Fi printer |
| 405 | 3D Printer #2    | octopi.home.lab   | 10   | 10.20.10.154 | Wired OctoPi instance |
| 406 | Raspberry Pi NUT | nut.home.lab      | 10   | 10.20.10.50  | Monitors UPS, signals safe shutdowns |

---

## Home Automation & IoT (VLAN 20)
| ID  | Device        | FQDN                | VLAN | IP           | Notes |
|-----|---------------|---------------------|------|--------------|-------|
| 500 | Philips Hue   | hue.home.lab        | 20   | 10.20.20.10  | Wired hub |
| 501+| IoT Devices   | deviceX.iot.home.lab| 20   | DHCP pool    | Smart plugs, bulbs, sensors etc. |

---

## LLM (Standalone for now)
| ID  | Device  | FQDN          | VLAN | IP           | Notes |
|-----|---------|---------------|------|--------------|-------|
| 600 | LLM Box | llm.home.lab  | 10   | 10.20.10.30  | Independent physical server, runs LLM models |

---

## Sandbox & Lab
| VMID | Name        | FQDN              | VLAN | IP           | Memory | Notes |
|------|-------------|------------------|------|--------------|--------|-------|
| 9000 | debian-lab  | lab1.home.lab    | 40   | 10.20.40.10  | 4 GB   | General Linux sandbox |
| 9001+| future-labs | labX.home.lab    | 40   | 10.20.40.x   | as req | Pentesting, experiments |

---

## VLAN Plan
| VLAN | Subnet         | Purpose            |
|------|----------------|--------------------|
| 10   | 10.20.10.0/24  | Main LAN (infra, apps, NAS, consoles, PCs, wired IoT) |
| 20   | 10.20.20.0/24  | IoT / Home Assistant (segregated Wi-Fi devices + hubs) |
| 30   | 10.20.30.0/24  | Guest Wi-Fi        |
| 40   | 10.20.40.0/24  | Lab / Sandbox      |
