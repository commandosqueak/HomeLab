# Proxmox VE Host Update (Manual, Safe)

> Purpose: update the **Proxmox host only** (not guests) in a controlled way.  
> Frequency: **Monthly** or when security fixes land.  
> Scope: Single-node today; cluster notes included for future.

## Pre-flight (5 min)

- ✅ **Backups OK:** Last PBS job green (or run one now).  
- ✅ **Guest safety point:** Run your weekly maintenance script (snapshots) or take a quick note of what’s running.  
- ✅ **Free space:** `df -h /boot /` (aim for a few GB free for kernels).  
- ✅ **Low load:** `uptime`, finish heavy tasks first.  
- ✅ **Console access available** (NoVNC/iKVM) in case SSH drops.

## 1) Check repo channels

GUI: **Datacenter → your-node → Updates → Repositories**  
- Ensure **No-subscription** repo is enabled (for homelab):  
  - `deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription`  
- (Optional) Disable Enterprise repo if present (requires subscription).  
- Click **Refresh** to verify no errors.

> CLI equivalent (view): `cat /etc/apt/sources.list /etc/apt/sources.list.d/*.list`

## 2) Preview what will change

```bash
sudo apt update
sudo apt list --upgradable
pveversion -v
```

## 3) Update the host

```bash
sudo apt -y dist-upgrade
pveam update   # optional: update container template metadata
```

Notes:
- If a **new kernel** was installed, a **reboot is required** to use it.
- The upgrade will **not** reboot guests. They keep running.

## 4) Reboot window

```bash
uname -r      # see current kernel
sudo reboot
# after boot
uname -r
pveversion -v
```

## 5) Post-checks

- **Web UI reachable:** `https://<host>:8006`
- **Storage OK:** Datacenter → Storage (no red).  
- **Guests running:** Summary → CPU/Mem graphs.  
- **ZFS (if used):** `zpool status` (no degraded pools).

## 6) Clean up old kernels (optional)

Keep at least **one** previous kernel until you’re happy.

```bash
dpkg -l | grep pve-kernel
sudo apt remove --purge pve-kernel-<old-version>
sudo apt -y autoremove --purge
```

## 7) Rollback plan

- Check `journalctl -p 3 -xb` and `/var/log/syslog`.  
- See what changed: `grep " install \| upgrade " /var/log/dpkg.log | tail -100`  
- In a pinch, boot the previous kernel from GRUB (older `pve-kernel-*`).

## Optional niceties

### Pin the previous kernel temporarily
```bash
dpkg -l | grep ^ii | grep pve-kernel
sudo apt-mark hold pve-kernel-<version>
# later:
sudo apt-mark unhold pve-kernel-<version>
```

### Microcode (CPU security/errata)
```bash
sudo apt -y install intel-microcode amd64-microcode || true
```

### Guest tools
- **QEMU agent** in VMs:  
  `sudo apt -y install qemu-guest-agent && sudo systemctl enable --now qemu-guest-agent`  
  In Proxmox: VM → **Options → QEMU Guest Agent = Enabled**.

## Cluster notes (for later)

- Update **one node at a time**.  
- **Migrate** movable VMs off the node you’re updating.  
- Reboot that node, verify healthy, repeat for the next.  
- Avoid live updates to **Corosync** across nodes simultaneously.

## Quick checklist

- [ ] `apt update && apt list --upgradable` reviewed  
- [ ] `apt dist-upgrade` completed  
- [ ] Rebooted; `uname -r` shows new kernel  
- [ ] UI/storage/guests healthy  
- [ ] (Optional) old kernels cleaned  
- [ ] Notes added to **Diary/YYYY-MM-DD**
