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


# 2025-10-03

## Progress
- **Pi-hole (CT 230):** Installed and working. Wired to **Unbound (recursive)** on `127.0.0.1#5335`. Pi-hole DNSSEC disabled (Unbound validates). Gravity updated.
- **Home Assistant (VM 220):** Created (`homeassistant`, 2 vCPU, 6 GB RAM, 100 GB). Onboarding in progress; migration from Pi planned via full backup.
- **DuckDNS:** Up and running for dynamic DNS.
- **WireGuard (CT 233):** Created (unprivileged, TUN+keyctl). `wg0` configured; **WGDashboard** installed and login fixed. Dashboard on `http://192.168.8.184:10086` (LAN-only).
- **Passwords:** Stored in Bitwarden (cloud) for now; plan to self-host Vaultwarden later.

## Notes
- WGDashboard password issue was INI parsing—symbols can be problematic. Using a strong password without `#` or `;` works reliably.
- No peers yet; add **phone peer (10.6.0.2/32)** via dashboard when ready.
- Keep dashboard LAN-only. Do **not** port-forward 10086.

## Next Steps (tomorrow / weekend)
- OPNsense prep: buy/install 2nd NIC, then install OPNsense.
- Move services to target subnets once VLANs exist:
  - Pi-hole → **10.20.10.21** (VLAN 10)
  - WireGuard CT → **10.20.10.24** (VLAN 10)
  - Home Assistant → **10.20.20.20** (VLAN 20 / IoT)
- OPNsense DHCP: set **Option 6** to Pi-hole IP.
- Open **UDP 51820** on OPNsense and restrict `WG_NET (10.6.0.0/24)`.
- Consider migrating HA from Pi via full backup; attach USB radios to VM.
- Add runbooks to repo (Plex, Vaultwarden) when you’re ready; hold off to save RAM.

## Quick Wins
- Add one mobile peer in WGDashboard and test handshake + DNS via Pi-hole.
- Snapshot Pi-hole, WireGuard CTs, and HA VM baselines in Proxmox.

# 🗓️ Daily Diary — 2025-10-05

## Summary
Big win. Reinstalled Proxmox cleanly, reattached VM storage, restored configs, and verified all core services. Lab is back online.

## What I did
- Reinstalled **Proxmox VE 8 (bookworm)** on NVMe boot disk.
- Mounted 1TB **VM-Storage** (`/mnt/pve/VM-Storage`) and made it persistent via `/etc/fstab`.
- Verified backups present: `pve-configs.tgz`, `host-scripts.tgz`, `cron*/`, `etc-ssl/`.
- Restored configs:
  - Extracted `pve-configs.tgz` → copied into node paths under `/etc/pve/nodes/<hostname>/{qemu-server,lxc}`.
  - Restarted `pve-cluster`, `pvedaemon`, `pveproxy`.
- Brought guests up, one-by-one, confirmed dashboards load:
  - **Apps VM (Docker)** — Heimdall, Portainer, Linkding, Homebox, LanguageTool, Uptime Kuma, DuckDNS ✔︎
  - **Home Assistant VM** ✔︎
  - **Pi-hole LXC** ✔︎
  - **WireGuard LXC** (WGDashboard accessible) ✔︎
- Wrote/added runbooks & cheat sheets:
  - `proxmox-clean-reinstall.md`
  - `proxmox-restore.md`
  - `restore-configs.md`
  - `fstab-recovery.md`
  - `disaster-recovery-lessons.md`

## Issues / Notes
- SSH/SCP timeouts earlier—bypassed by placing backups on the VM-Storage disk.
- Proxmox service reload took a while; resolved after proper config placement + restart.
- NVIDIA drivers abandoned, will install Plex bare metal instead

## Next actions (short-term)
- Confirm static DHCP reservations for: Proxmox, Apps VM, Pi-hole, WG, HA.
- Snapshot each VM/LXC now that they’re healthy.

## Next actions (mid-term)
- OPNsense build when the new NIC arrives (initially flat LAN; VLANs later with managed switch & AP).
- Consider **PBS + offsite backups**.
- Add **Watchtower** for container updates and finalize maintenance script cadence.

## Mood / Energy
- Tired but relieved. Ruby approved (eventually). 🐾


# Rebuild Diary — Proxmox & OPNsense Bring‑Up
_Date: 2025-10-05 19:45:36_

## TL;DR
- **Fresh PVE install (Debian 13 / Proxmox Kernel 6.14.11-3-pve)** on M.2 system disk.
- **VM-Storage (ext4) mounted persistently** via `/etc/fstab` using UUID `d895a34f-621c-41f7-97f6-a1202124fbf0` → `/mnt/pve/VM-Storage`.
- **Configs restored** (QEMU/LXC + basic network), then **storage re-added in PVE**.
- All previous **VMs/CTs back up**.
- Installed new **2.5GbE NIC** (interface `ens2`) and created **vmbr1** for WAN.
- **OPNsense VM (ID 300)** installed from the VGA **.img** (not ISO), LAN: `192.168.8.10/24` (vtnet0 on `vmbr0`), WAN: vtnet1 on `vmbr1` (no carrier yet).
- Created a **compressed raw snapshot** of the PVE root LV after OPNsense install:
  - `/mnt/pve/VM-Storage/snapshots/root-post-opnsense.gz` (≈3.4 GiB)
  - `sha256: 3d2df3746cc61eff572d0b34501de4611576b7d616f3c2db81c1a192515318ee`
- **Open items:** bring WAN link up (connect/bridge to internet), finish OPNsense wizard, test outbound.

---

## System facts
- **Hostname:** `proxmox`
- **PVE IP:** `192.168.8.2/24` on **vmbr0** (bridged to `enp4s0`)
- **NICs:** `enp4s0` (1 GbE), `ens2` (2.5 GbE)
- **Bridges:**
  - `vmbr0` ↔ `enp4s0` (LAN / management)
  - `vmbr1` ↔ `ens2` (WAN for OPNsense)
- **Storage:** `/mnt/pve/VM-Storage` (ext4) — contains `images/`, `template/iso/`, `pve-backups/`, `snapshots/`, etc.

---

## What we did — timeline
1. **Confirmed backups present** on `VM-Storage:/pve-backups` (cron/, etc-ssl/, `pve-configs.tgz`, `host-scripts.tgz`).  
2. **Reinstalled PVE** (Debian 13/trixie base, Proxmox kernel 6.14.11-3-pve).  
3. **Mounted VM datastore** persistently:
   - Added to `/etc/fstab`:
     ```
     UUID=d895a34f-621c-41f7-97f6-a1202124fbf0  /mnt/pve/VM-Storage  ext4  defaults  0  2
     ```
   - `systemctl daemon-reload && mount -a` → verified content.  
4. **Restored platform configs:**
   - Extracted `pve-configs.tgz` into `/pve-backup/` then copied back:
     - `/pve-backup/qemu/*.conf` → `/etc/pve/qemu-server/`
     - `/pve-backup/lxc/*.conf`  → `/etc/pve/lxc/`
     - `/pve-backup/sys/storage.cfg` → `/etc/pve/storage.cfg` (also re-added manually later)
     - (Hosts/interfaces reviewed; PVE services restarted)
   - Restarted: `pveproxy`, `pvedaemon`, `pve-cluster`.  
5. **Re-added storage “VM-Storage”** (we initially forgot which blocked VM starts):
   - `pvesm add dir VM-Storage /mnt/pve/VM-Storage`  
   - Afterwards **all VMs/CTs started OK**.  
6. **Created safety snapshots:**
   - LVM snapshots of root (pre‑experiments).  
   - Later took a **raw dd snapshot** after OPNsense install (see _Backups_ below).  
7. **Installed new 2.5GbE NIC** → appeared as `ens2`.  
8. **Configured `vmbr1`** for WAN:
   ```
   auto vmbr1
   iface vmbr1 inet manual
       bridge-ports ens2
       bridge-stp off
       bridge-fd 0
   ```
   - Restarted networking; verified with `brctl show`:
     - `vmbr0: enp4s0 (+ tap* for VMs)`
     - `vmbr1: ens2 (+ tap300i1 for WAN)`  
9. **Built OPNsense VM (ID 300):**
   - **Disks:** `scsi0` 32 GiB (qcow2) on `VM-Storage`.
   - **Networks:** `net0` virtio on `vmbr0` (LAN), `net1` virtio on `vmbr1` (WAN).
   - **Media:** Upload **OPNsense-25.7-vga-amd64.img** to `local:iso/` and **import as a disk**:
     ```bash
     qm importdisk 300 /var/lib/vz/template/iso/OPNsense-25.7-vga-amd64.img VM-Storage --format raw
     qm set 300 --scsi1 VM-Storage:300/vm-300-disk-1.raw
     qm set 300 --boot order=scsi1
     ```
   - **Install:** ZFS (single‑disk), accepted 8 GiB swap, set root password, rebooted.
   - **Switch boot back to system disk** and detach installer disk:
     ```bash
     qm set 300 --boot order=scsi0
     qm set 300 --delete scsi1
     ```
   - **Result:** LAN `vtnet0` = `192.168.8.10/24`, WAN `vtnet1` = bridged to `vmbr1`.  
   - **Web UI reachable** at `https://192.168.8.10` (after ensuring `tap300i0/1` attached to bridges).  
10. **Wizard basics:** timezone Europe/London, dark mode, DNS 1.1.1.1/8.8.8.8 (no override by DHCP).  
11. **WAN pending:** `ens2` currently **no carrier** (not cabled to an upstream that provides IP). **Ping to 8.8.8.8 fails** until WAN is wired/online.

---

## Backups & snapshots
- **Post-OPNsense root image (compressed):**
  - Path: `/mnt/pve/VM-Storage/snapshots/root-post-opnsense.gz`
  - Size: ~3.4 GiB (from ~69 GiB raw read)
  - Created with:
    ```bash
    dd if=/dev/pve/root | gzip -1 > /mnt/pve/VM-Storage/snapshots/root-post-opnsense.gz
    sha256sum /mnt/pve/VM-Storage/snapshots/root-post-opnsense.gz
    gzip -t /mnt/pve/VM-Storage/snapshots/root-post-opnsense.gz
    ```
  - SHA256: `3d2df3746cc61eff572d0b34501de4611576b7d616f3c2db81c1a192515318ee`
- **LVM snapshots (earlier safety points):**
  - `lvcreate -L2G -s -n root-pre-nvidia /dev/pve/root`
  - `lvcreate -L2G -s -n root-pre-nvidia2 /dev/pve/root`

> _Recommendation_: Keep the `.gz` snapshot **read‑only** and optionally replicate it to offline storage or the NAS for belt‑and‑braces.

---

## Current state
- PVE up, VMs/CTs running as before.
- OPNsense VM running; **LAN OK** at `192.168.8.10`.
- WAN bridge configured (`vmbr1` ↔ `ens2`) but **no internet** yet (link down).

---

## Next steps (when you’re fresh)
1. **Physically wire WAN (`ens2`)** to a live upstream:
   - If double‑NAT behind your current router: plug `ens2` into a LAN port on router and set **WAN to DHCP**.
   - If using a modem in bridge mode: connect modem → `ens2` and configure **PPPoE/static** as appropriate.
2. In OPNsense:
   - **Interfaces → WAN**: set IPv4 to **DHCP** (or PPPoE/static), IPv6 as needed.
   - **System → Routing → Gateways**: verify a default gateway appears and is **Online**.
   - **Firewall → NAT → Outbound**: start with **Automatic**.
   - **Diagnostics → Ping**: test `8.8.8.8` and `opnsense.org`.
3. **Protect backups**:
   - `chattr +i /mnt/pve/VM-Storage/snapshots/root-post-opnsense.gz` (optional, if filesystem supports it).
   - Copy snapshot to second location/NAS.
4. **Housekeeping**:
   - Tag VM 300 “OPNsense” with note `LAN: 192.168.8.10/24 (vmbr0), WAN: vmbr1`.
   - Consider enabling Proxmox **backups** to a `dump/` schedule for OPNsense.

---

## Handy references
- **PVE Bridges:**
  ```bash
  brctl show
  # vmbr0: enp4s0, tap... (LAN)
  # vmbr1: ens2, tap300i1 (WAN)
  ```
- **Interfaces (short):**
  ```bash
  ip -br addr
  ethtool ens2   # expect "Link detected: yes" when cabled
  ```
- **OPNsense VM ID:** `300`
- **LAN access:** `https://192.168.8.10`

---

## Notes
- We explicitly avoided NVIDIA driver shenanigans today; Plex HW transcode plan moved to a different host to keep PVE stable.
- Ruby supervised multiple times 🐾 — purr‑powered ops were successful.

# 2025-10-06 — Rebuild, OPNsense VM online, and alerting set up

## TL;DR
- Fully reinstalled **Proxmox VE** on the M.2 SSD and restored configuration from the SATA SSD.
- Re-mounted `VM-Storage`, re-added it as a PVE storage, and all VMs/LXCs booted.
- Created **protected snapshots** (LVM + raw dd image) so we can roll back fast.
- Installed a new **2.5 GbE NIC**, bridged it as `vmbr1`, and brought up an **OPNsense** VM with working LAN/WAN vtnet NICs.
- Fixed **PVE Firewall** rules and enabled **email alerts** (Postfix → Gmail), **SMART** and **ZFS ZED** notifications.
- Enabled **SSH keys** + **TOTP** for `root@pam` and `ellie@pam`.

---

## What we did (chronological highlights)

### 1) Fresh Proxmox install & base checks
- Installed PVE (Debian 13 / kernel `6.14.11-3-pve`) on the M.2 (system) disk.
- Verified host networking: `vmbr0` on `192.168.8.2/24`.

### 2) Mount the VM data disk and persist it
- Confirmed UUID of the SATA SSD (VM datastore):  
  `UUID="d895a34f-621c-41f7-97f6-a1202124fbf0" (ext4)`
- Added to `/etc/fstab` and mounted at `/mnt/pve/VM-Storage`.
- Verified content (backups, images, etc.) is present after reboot.

### 3) Restore Proxmox configs
- Extracted `pve-configs.tgz` and copied into place:
  - `/etc/pve/storage.cfg`
  - `/etc/network/interfaces` (careful merge)
  - `/etc/hosts` (FQDN `proxmox.home.lab`)
  - VM/CT configs into `/etc/pve/qemu-server/` and `/etc/pve/lxc/`
- Restarted key services (`pveproxy`, `pvedaemon`, `pve-cluster`) and validated the GUI.
- **Storage**: Re-added `VM-Storage` (Directory, path `/mnt/pve/VM-Storage`, content `Disk image, ISO, VZDump, Snippets`).

### 4) Snapshots / rollbacks
- LVM snapshots on root:
  - `lvcreate -L2G -s -n root-pre-nvidia /dev/pve/root` (kept for history)
  - `lvcreate -L2G -s -n root-post-opnsense /dev/pve/root`
- Immutable raw backup image:
  - `dd if=/dev/pve/root | gzip -1 > /mnt/pve/VM-Storage/snapshots/root-post-opnsense.gz`
  - Verified with `gzip -t` and `sha256sum`.

### 5) New NIC install & bridges
- Confirmed new NIC as `ens2`.
- Created `vmbr1` bridged to `ens2` (no IP on host):
  ```text
  auto vmbr1
  iface vmbr1 inet manual
      bridge-ports ens2
      bridge-stp off
      bridge-fd 0
  ```
- Verified bridges & tap attachments with `brctl show`.

### 6) OPNsense VM creation & install
- Uploaded OPNsense installer image to `local` storage (`/var/lib/vz/template/iso`).
- Created VM **300**:
  - CPU `host`, 2 vCPU, 4 GiB RAM
  - Disks on `VM-Storage` (virtio-scsi)
  - `net0` → `vmbr0` (LAN), `net1` → `vmbr1` (WAN)
- Installed OPNsense (ZFS + 8 GiB swap), set **LAN IP 192.168.8.10/24**, WAN on vmbr1.
- Removed installer ISO and set boot order → disk.
- **Fix that made the Web UI appear**: ensured both VM NICs are attached to the correct bridges *and* the `vmbr1` bridge carried the physical `ens2`. After that, the GUI was reachable on `https://192.168.8.10`.

### 7) Email alerts & notifications
- Outbound mail via Gmail SMTP (Postfix SASL) — fixed credentials & tested.
- Set `root@pam` email:
  ```bash
  pveum user modify root@pam --email "eleanormhathaway@gmail.com"
  ```
- Datacenter → Notifications:
  - Target: Sendmail → recipient = your Gmail.
  - Matcher: severity `warning,error` → target above.
- `/etc/aliases` → `root: your.email@example.com` + `newaliases`.
- **SMART**:
  ```bash
  apt install -y smartmontools
  # DEVICESCAN with -m your email -M daily
  systemctl enable --now smartd
  ```
- **ZFS ZED** (if/when ZFS used):
  - `ZED_EMAIL_ADDR="your.email@example.com"` → `systemctl enable --now zfs-zed`.

### 8) Access & hardening
- **SSH**: Key-based auth for `ellie` (ed25519), correct perms on `~/.ssh` + `authorized_keys`.
- **2FA (TOTP)**: Enabled for `root@pam` and `ellie@pam` in GUI. Tested PVE login.
- **PVE Firewall**:
  - Fixed cluster rules to correct syntax (uppercase `ACCEPT`, correct section).
  - Allowed 8006/tcp from admin subnet, 22/tcp for SSH, and ICMP.

---

## Working state at end of day
- Proxmox host: ✅ up, GUI reachable on `https://192.168.8.2:8006`
- VM datastore: ✅ mounted and configured as `VM-Storage`
- VMs/LXCs: ✅ started and dashboards reachable
- OPNsense VM (300): ✅ installed, LAN `192.168.8.10/24`, Web UI reachable
- New 2.5 GbE NIC: ✅ bridged as `vmbr1` for OPNsense WAN
- Email alerts: ✅ working (PVE notifications, SMART/ZED hooks in place)
- SSH & MFA: ✅ keys + TOTP working for `root@pam` and `ellie@pam`
- Rollback artifacts: ✅ LVM snapshot + 3.4 GiB gzipped raw image saved

---

## Notes & gotchas captured
- If OPNsense GUI is unreachable after install, double-check:
  - VM NIC → correct bridge mapping (`vmbr0` LAN, `vmbr1` WAN).
  - `vmbr1` must actually include the physical `ens2`.
  - Boot order set to disk (remove installer ISO).
- PVE Firewall cluster rules require correct **section** and **syntax**; “accept” vs `ACCEPT` matters.
- Avoid mixing Debian releases — pinning and repositories were corrected (stays on Bookworm for PVE 8).

---

## Next actions (planned for weekend)
- Finish OPNsense bootstrap: WAN DHCP/static, LAN DHCP server, admin ACLs, DNS, NTP.
- Cut-over plan & maintenance window (switching LAN to OPNsense, router changes).
- Optional: import old VMware lab VMs later (via `qm importdisk`/conversion), when RAM allows.
- Documentation tidy-up (folders for “Runbooks”, “Reference”, “Recovery”).

