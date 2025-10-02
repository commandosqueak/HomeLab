# Homelab Runbook

## Chapter 1 – Proxmox VE Installation & Basic Setup

### 1. Prep & BIOS
- **Enable virtualization:** VT-x/AMD-V + IOMMU/SR-IOV.
- **Boot order:** USB first, SSD second.
- Disable Secure Boot if enabled.

### 2. Boot Media
- Grab the latest ISO: [https://www.proxmox.com/en/downloads](https://www.proxmox.com/en/downloads)
- Verify checksum.
- Use Ventoy (already on your USB) and copy the ISO.

### 3. Install
1. Boot the old gaming PC from the Ventoy USB.
2. Pick the Proxmox ISO.
3. Target drive: *dedicated SSD for Proxmox* (ZFS recommended for snapshots/RAID-Z if you add disks later).
4. Set root password + email.
5. Network:
   - Static IP: `10.20.10.2`
   - Gateway: `10.20.10.1` (OPNsense later)
   - DNS: `10.20.10.53` (Pi-hole, or your ISP until Pi-hole is up)
6. Finish install and reboot.

### 4. First Login
- Browser: `https://10.20.10.2:8006`
- User: `root`
- Password: (the one you set)

### 5. Post-Install
- Update: `apt update && apt dist-upgrade`
- Add Proxmox VE repo keys (non-subscription if you don’t have a sub).
- Create a ZFS storage pool if not done during install.
- Set backups target (NAS).

### 6. Next Steps
- Create a Linux VM template for fast cloning.
- Plan VLANs in Proxmox bridges (vmbr0 for Main LAN, additional vmbrs for IoT/Guest/Lab).

---

## Chapter 2 – OPNsense Installation & Basic Setup

### 1. VM Creation
- In Proxmox: `Create VM → OPNsense`
- CPU: 2–4 cores, RAM: 4 GB+.
- Disks: 20 GB+ (virtio).
- Add **two NICs**:
  - `vmbr0` (LAN)
  - `vmbr1` (WAN)

### 2. Install OPNsense
- Boot from OPNsense ISO.
- Accept defaults, set strong root password.

### 3. Initial Config
- Assign interfaces: WAN to `vmbr1`, LAN to `vmbr0`.
- Set LAN IP: `10.20.10.1/24`.
- Enable DHCP only for WLAN/guest VLANs if desired.

### 4. Security & Updates
- Web UI: `https://10.20.10.1`
- Change admin password.
- Update firmware to latest.

### 5. Core Services
- **Firewall rules:** allow LAN → WAN, restrict VLANs as planned.
- **Dynamic DNS (DuckDNS):**
  - Install the `os-ddclient` plugin.
  - Set provider = DuckDNS, token & domain.
- **DNS:** point to Pi-hole for ad-blocking.
- Optional: Enable IPS/IDS (Suricata) if needed.

---

**Tip:** Keep a copy of this runbook and every config file in Git.
Next chapter will cover VM provisioning (Pi-hole, Home Assistant, etc.).
