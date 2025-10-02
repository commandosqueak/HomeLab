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
- ✅ SSH into `apps (200)` as **ellie** via MobaXterm; copy/paste sorted.
- ✅ Installed **qemu-guest-agent** on `apps` (Proxmox now shows IP).
- ✅ Deployed containers:
  - Portainer (fixed env to **Docker local** via `/var/run/docker.sock`)
  - Heimdall
  - Linkding (set superuser via env; changed password after login)
  - Uptime Kuma
  - Homebox
  - LanguageTool (API up on :8010)
- ✅ Obsidian vault is now the **GitHub repo**; committed & pushed changes.
- ✅ Added README seed files so folders show on GitHub.

### Notes
- `apps` currently on **DHCP 192.168.8.180**; will become **10.20.10.20** after OPNsense.
- LanguageTool extension “Local server” wants **localhost**; will tunnel or switch to Traefik later.
- Git/GitHub: Windows is case-insensitive; GitHub isn’t. Folder case (`Docs` vs `docs`) needs a two-step rename to reflect online.

### Issues / Gotchas
- Initial Portainer wizard picked Podman → snapshot error; removed Podman env, added **Docker → Local**.
- GitHub Desktop kept stale `origin` after remote deletion → removed remote, republished.
- Empty folders don’t show on GitHub → added `README.md` in each.

### Next Steps (tomorrow)
- [ ] Rename `Docs → docs` and subfolders properly (two-step rename: `Docs → docs_tmp → docs`), then commit/push.
- [ ] Move/runbooks into `docs/runbook/` (Portainer, Heimdall, Linkding, Kuma, Homebox, LT, Vaultwarden).
- [ ] Add `docs/runbook/README.md` index and update links if casing changed.
- [ ] (Optional quick win) Deploy **Vaultwarden** on port 8081 (compose ready).
- [ ] Prepare **Traefik** plan (after OPNsense): HTTPS + hostnames.
- [ ] Post-network: set static IPs + Pi-hole/OPNsense DNS A-records for services.
- [ ] Later hardening: SSH keys everywhere; disable password auth.

### Tiny Wins to Celebrate
- Six containers live in one evening 🎉
- Repo + Obsidian finally in sync.
- Guest agent installed—no more “what’s my IP?” guessing.

