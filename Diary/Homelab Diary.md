## 2025-09-28
### Progress
- Planned homelab rebuild after repurposing old gaming PC.
- Decided to install **Proxmox VE** as the hypervisor.
- Wiped SSDs with GParted to prepare for fresh installs.
- Initial runbook structure drafted (Proxmox Chapter 1, OPNsense Chapter 2).

### Notes
- Using ZFS for drive layout.
- Secure Boot disabled for Proxmox compatibility.
- Networking goal: 2.5Gb to devices, 10Gb SFP+ backbone between router, NAS, switch, and server.

---

## 2025-09-29
### Progress
- Chose **OPNsense** over pfSense (UI, Suricata IDS, flexibility).
- Decided to add **Pi-hole** for DNS/ad-blocking.
- VLAN design planned:
  - VLAN 10: Main LAN
  - VLAN 20: IoT/Home Automation
  - VLAN 30: Guest
  - VLAN 40: Lab/Sandbox
- Created initial network diagram with trunk lines between switches.

### Notes
- Need managed switches for VLAN support (unmanaged won’t cut it).
- Considering cheap managed switches with 10Gb uplinks + 2.5Gb downlinks.

---

## 2025-09-30
### Progress
- Added more services to planned stack:
  - **Home Assistant**, **Heimdall**, **Linkding**, **Homebox**, **LanguageTool**, **Uptime Kuma**, **Portainer**.
- Documented backup plan: **Veeam → NAS → Cloud**.
- Decided to track everything in Git/Obsidian, with custom VM/App IDs.
- Finalized **hybrid ID scheme** (VMIDs for Proxmox, AppIDs nested for Docker containers).
- Created runbook entries for each VM and container with VLANs + IPs.

### Notes
- Started thinking about custom backgrounds per VM/app to make them visually distinct.
- Planned Raspberry Pi with NUT to manage UPS events.
- Documented plan to P2V (physical-to-virtual) the LLM in future.

---

## 2025-10-01
### Progress
- Updated runbook with hybrid **ID scheme** (VMIDs + AppIDs).
- Added extra devices (consoles, 3D printers, Hue hub, NUT Pi) into Markdown + diagram.
- Created **`debian-lab (9000)`** VM for sandbox/pentesting (VLAN 40).
- Created **Docker VM (`apps`, 200)**:
  - 50 GB disk, 4 GB RAM, 2 vCPU, DHCP networking (temporary).
  - Hostname set to `apps`.
- Installed **Docker CE + Compose** from official Docker repo:
  - Fixed malformed sources.list entry.
  - Imported GPG key properly.
  - Verified `docker --version` and `docker compose version`.
- Ready for first test stack (Portainer/Heimdall) tomorrow.

### Notes
- OPNsense not yet deployed (waiting on extra NIC).
- Docker VM left on DHCP until OPNsense controls LAN.
- Lab diary started today to track incremental progress.

### Next Steps
- Deploy a small test stack (Portainer or Heimdall) on `apps (200)`.
- Install OPNsense once new NIC arrives.
- Assign static IPs/DNS (10.20.10.x) after OPNsense is live.

## 2025-10-02
### Progress
- SSH working to `apps (200)` via MobaXterm as user **ellie** (password auth for now).
- Deployed containers:
  - **Portainer** (https://192.168.8.180:9443) — fixed environment to **Docker / local (unix:///var/run/docker.sock)**.
  - **Heimdall** (http://192.168.8.180:8080).
  - **Linkding** (http://192.168.8.180:9090) — superuser defaults set via compose; changed after first login.
  - **Uptime Kuma** (http://192.168.8.180:3001).
  - **Homebox** (http://192.168.8.180:7745).
  - **LanguageTool** (http://192.168.8.180:8010) — extension set to “Local server” (remote routing TBD).
- Obsidian plugins installed: Tasks, Editor Width Slider, Beautitab, Callout Manager, Templater, Dataview, Advanced Tables, Calendar, Kanban (installed), Excalidraw.
- Finished setting up git desktop, online and used my obsidian vault as my repo.

### Notes
- Proxmox Summary still shows “guest agent not running” on `apps` — install `qemu-guest-agent` later.
- `apps` VM currently on **DHCP** (192.168.8.180). Will switch to **10.20.10.20** after OPNsense is live.
- LanguageTool browser extension doesn’t take remote IP in “Local server” mode; will use SSH tunnel or Traefik + custom URL later.
- All service passwords stored in **Bitwarden Cloud**; planning **Vaultwarden** self-hosted for lab.

### Next Steps
- [ ] Install **qemu-guest-agent** on `apps` (and other VMs).
- [ ] Stand up **Vaultwarden** (self-hosted Bitwarden) on `apps` (port 8081 for now).
- [ ] Prepare **Traefik** (after OPNsense + static IP) for HTTPS + pretty hostnames.
- [ ] Set **static IP** for `apps` → `10.20.10.20` once OPNsense is in place.
- [ ] Add DNS A-records in Pi-hole/OPNsense for services (e.g., `portainer.container.home.lab`).
- [ ] Plan **backups** for stack volumes (NAS target; later cloud copy).
- [ ] Switch SSH to **key-based auth** and disable password login once stable.
- [ ] (Optional) DuckDNS updater container if DDNS not handled on OPNsense.


