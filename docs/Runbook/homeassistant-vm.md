# Home Assistant — VM Runbook (VMID 220)

**Goal:** Run Home Assistant OS (HAOS) as a Proxmox VM and (later) migrate from an existing Raspberry Pi instance.

- **VMID:** 220
- **Name/Hostname:** `homeassistant`
- **CPU/RAM:** 2 vCPU / 6 GB RAM (start; adjust later)
- **Disk:** 100 GB (VirtIO SCSI)
- **BIOS:** UEFI/OVMF
- **NIC:** `vmbr0` (untagged for now). Later set **VLAN Tag 20** (IoT).
- **IP (now → later):** DHCP → `10.20.20.20` (reserved)

---

## 1) Create VM 220 (helper script or manual)

### Using the Proxmox helper script (recommended)
- Select **Home Assistant OS (HAOS)**.
- Set **VMID 220**, **disk 100 GB**, **2 vCPU**, **6 GB RAM**, **UEFI/OVMF**.
- NIC on `vmbr0` (no VLAN tag yet).

### Manual (quick)
1. **Create VM** → **BIOS** = *OVMF (UEFI)*, **SCSI Controller** = *VirtIO SCSI*.
2. **Disk**: VirtIO SCSI, 100 GB (thin).
3. **Import HAOS disk** if using qcow2:  
   ```bash
   qm importdisk 220 /path/to/haos.qcow2 <proxmox-storage>
   ```
   Attach to VM 220 as SCSI, set *Boot*.
4. **CPU/RAM**: 2 vCPU, 6 GB RAM.
5. **NIC**: bridge `vmbr0`. Start VM.

---

## 2) First Boot & Onboarding

1. Wait for HAOS to **prepare** (can take 10–20 minutes).
2. Find the IP in Proxmox summary or DHCP leases.
3. Browse to `http://<ha-ip>:8123` → complete onboarding (create user, set timezone to **Europe/London**, etc.).
4. **Advanced Mode**: Profile (bottom-left) → toggle *Advanced Mode*.
5. **Hostname** (if needed): Settings → System → *Host* → **Change hostname** → `homeassistant`.

> Take a **Backup** (Settings → System → Backups → *Create*) once it boots—this is your **baseline** snapshot.

---

## 3) Migration from Raspberry Pi (later)

### A) Full backup on the Pi
- Pi HA → **Settings → System → Backups → Create backup** (include add-ons).
- Download the generated `.tar` to your PC.

### B) Restore on the VM
- If still on onboarding, use **Restore from backup** on the welcome screen.  
- If already onboarded: **Settings → System → Backups → Upload** `.tar` → **Restore** (Full).  
- HA will restart a few times—normal.

### C) USB radios (Zigbee/Z-Wave) move
1. **Shutdown** HA on the Pi cleanly.
2. Plug dongles into the **Proxmox host**.
3. Proxmox → VM 220 → **Hardware → Add → USB Device** → select the device.  
   *Tip: choose **Use USB Port** so it persists across reboots.*
4. In HA, integrations should reference devices under `/dev/serial/by-id/...` (stable path).

- **ZHA**: usually fine if the same `/dev/serial/by-id/...` is used—no re-pairing.
- **Zigbee2MQTT**: ensure `network_key` & `pan_id` migrate; full backup normally carries them.
- **Z-Wave JS**: network lives on the stick; verify the serial path and it should import cleanly.

### D) Post-restore sanity
- Users, dashboards, automations exist and run.
- Add-ons started (SSH, File Editor, Zigbee2MQTT/ZHA, etc.).
- Update any integrations that point to old IPs.
- Take a new **backup** (“post-restore baseline”).

### E) Retire the Pi
- Power down the Pi HA once the VM is confirmed working to avoid conflicts.

---

## 4) Networking: now vs later

**Now:** VM on flat LAN (`vmbr0`, untagged) via DHCP.  
**Later (IoT VLAN 20):**
1. Ensure switch port to Proxmox carries VLAN 20 (trunk).
2. Proxmox → VM 220 → **Hardware → Network Device → VLAN Tag = 20**.
3. In OPNsense, reserve **10.20.20.20** for HA MAC (DHCP).
4. Add DNS override: `homeassistant.iot.home.lab → 10.20.20.20`.
5. Firewall: allow only required ports from main LAN to VLAN 20 (8123, API ports you use).
6. For discovery across VLANs, use an **mDNS repeater** or set integrations by IP.

---

## 5) Add-ons & Quality of Life

- **SSH & Web Terminal (Official)** – shell access to HAOS.
- **File Editor** – quick edits to configuration.
- **Samba Share** – easy file access from Windows (optional).
- **MariaDB** – for Recorder if your history grows beyond SQLite.
- **Backups** – schedule regular snapshots; consider offloading to NAS.

---

## 6) Maintenance

- **Updates:** Settings → System → Updates (OS, Supervisor, Core). Apply separately if queued.
- **Snapshots:** Before major changes, take a backup. Keep “baseline” and “post-restore” copies.
- **Logs:** Settings → System → Logs → check after add-on changes.

---

## 7) Troubleshooting

- **Stuck on “Preparing”:** wait 20–30 mins; if still stuck, reboot the VM once. Check Proxmox storage & network.
- **No IP:** ensure VM NIC is bridged to correct `vmbr` and DHCP is available.
- **USB device not found:** re-add the USB device using “Use USB Port” and reboot the VM.
- **Integrations broken after restore:** update IP/hostnames or tokens; check **Configuration → Settings → Devices & Services** for reconfig prompts.

---

### Cut-over Checklist (paste into Diary when you migrate)
- [ ] Pi: Full backup created & downloaded  
- [ ] VM: Full restore completed (reboots OK)  
- [ ] USB dongles moved & attached to VM  
- [ ] Zigbee/Z-Wave OK via `/dev/serial/by-id/...`  
- [ ] Automations & add-ons working  
- [ ] Pi HA powered down  
- [ ] Post-restore baseline backup taken
