# Proxmox Clean Reinstall & Reattach Storage — Playbook

**Scenario:** Proxmox VE is messy (mixed repos / broken deps). You want a clean reinstall on the **boot NVMe** while keeping **all VMs/CTs** safe on the **separate 1 TB “VM-Storage”** disk.

**Outcome:** Fresh Proxmox on the 256 GB NVMe, reattach the 1 TB disk, restore guest configs, and you’re back.

---

## 0) Quick assumptions

- Boot disk: **256 GB NVMe** (target for reinstall).
- Guest storage disk: **1 TB SATA** mounted as **`/mnt/pve/VM-Storage`** (keep untouched).
- Storage type on 1 TB: **Directory** (qcow2/raw images under `images/<VMID>` and LXC under `subvol-<VMID>-disk-0`).
- You’ll run these steps **from the node console/NoVNC**.

---

## 1) Pre-flight: verify everything lives on `VM-Storage`

```bash
pvesm status
qm list
pct list
```

For each VM/CT (check in GUI or `*.conf` later):

- VM disks should be under:  
  `/mnt/pve/VM-Storage/images/<VMID>/vm-<VMID>-disk-*.qcow2`
- LXC rootfs under:  
  `/mnt/pve/VM-Storage/subvol-<VMID>-disk-0`

If anything is still on the boot disk, **stop guest** and **Move Disk** to `VM-Storage` first.

---

## 2) Save critical configs (tiny, priceless)

```bash
mkdir -p /root/pve-backup/{qemu,lxc,sys}
cp -a /etc/pve/qemu-server/*.conf /root/pve-backup/qemu/ 2>/dev/null || true
cp -a /etc/pve/lxc/*.conf         /root/pve-backup/lxc/  2>/dev/null || true
cp -a /etc/pve/storage.cfg        /root/pve-backup/sys/  2>/dev/null || true
cp -a /etc/network/interfaces     /root/pve-backup/sys/
cp -a /etc/hosts                  /root/pve-backup/sys/
tar -czf /root/pve-configs.tgz -C /root pve-backup
```

Also grab any host scripts (optional):

```bash
tar -czf /root/host-scripts.tgz /usr/local/sbin 2>/dev/null || true
```

**Copy** `/root/pve-configs.tgz` (and `host-scripts.tgz` if created) **off the box** to your PC/NAS (SCP/WinSCP).

> Optional belt-and-braces:  
> `vzdump <VMIDs/CTIDs> --mode stop --compress zstd --dumpdir /mnt/backup`

Shut down guests when done.

---

## 3) Reinstall Proxmox VE (clean, Bookworm)

1. Boot latest **Proxmox VE 8.x ISO**.
2. **Select only the 256 GB NVMe** for installation. **Do not** touch the 1 TB disk.
3. Finish wizard (hostname/IP/timezone).
4. First boot: log in (console or `https://host:8006`).

Add repos (no-subscription + Debian bookworm):

```bash
cat >/etc/apt/sources.list.d/proxmox.sources <<'EOF'
Types: deb
URIs: http://download.proxmox.com/debian/pve
Suites: bookworm
Components: pve-no-subscription
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
EOF

cat >/etc/apt/sources.list.d/debian.sources <<'EOF'
Types: deb
URIs: http://deb.debian.org/debian
Suites: bookworm bookworm-updates
Components: main contrib non-free non-free-firmware
Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg

Types: deb
URIs: http://security.debian.org/debian-security
Suites: bookworm-security
Components: main contrib non-free non-free-firmware
Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg
EOF

apt update && apt -y full-upgrade
```

---

## 4) Reattach the 1 TB disk (non-destructive)

Identify and mount:

```bash
lsblk -f
# assume 1TB is /dev/sdb1 and ext4 (adjust if xfs/btrfs)
mkdir -p /mnt/pve/VM-Storage
mount /dev/sdb1 /mnt/pve/VM-Storage
```

Persist via `/etc/fstab`:

```bash
blkid /dev/sdb1
# copy the UUID, then:
nano /etc/fstab
# Add a line (ext4 example):
# UUID=<THE-UUID>  /mnt/pve/VM-Storage  ext4  defaults  0  2

mount -a
```

Add storage to Proxmox with the **same ID** as before:

- GUI: **Datacenter → Storage → Add → Directory**
  - **ID:** `VM-Storage`
  - **Directory:** `/mnt/pve/VM-Storage`
  - **Content:** `Disk image, Container, ISO, Snippets` (as needed)
- Or CLI:
  ```bash
  pvesm add dir VM-Storage --path /mnt/pve/VM-Storage --content images,rootdir,iso,snippets
  ```

You should already see folders like `images/<VMID>/` and/or `subvol-<VMID>-disk-0` on that disk.

---

## 5) Restore guest configs

Copy your tarball back and place configs:

```bash
scp <yourpc>:/path/to/pve-configs.tgz /root/
tar -xzf /root/pve-configs.tgz -C /root

cp -a /root/pve-backup/qemu/*.conf /etc/pve/qemu-server/ 2>/dev/null || true
cp -a /root/pve-backup/lxc/*.conf  /etc/pve/lxc/         2>/dev/null || true
```

List guests:

```bash
qm list
pct list
```

If any VM shows a missing disk, open its config `/etc/pve/qemu-server/<VMID>.conf` and fix disk lines to match the real paths on `VM-Storage`, e.g.:

```
scsi0: VM-Storage:images/100/vm-100-disk-0.qcow2
```

CLI fix example:

```bash
qm set 100 --scsi0 VM-Storage:images/100/vm-100-disk-0.qcow2
```

For LXCs, ensure `rootfs` matches an existing subvol:

```bash
pct set 230 -rootfs VM-Storage:subvol-230-disk-0
```

Start guests **one by one**:

```bash
qm start 100
pct start 230
```

---

## 6) Restore extras (optional)

- Host scripts:
  ```bash
  scp <yourpc>:/path/to/host-scripts.tgz /root/
  tar -xzf /root/host-scripts.tgz -C /
  chmod +x /usr/local/sbin/*.sh 2>/dev/null || true
  ```
- Cron jobs (weekly maintenance):
  ```bash
  crontab -e
  # 15 3 * * 0 /usr/local/sbin/lab-maintenance.sh >> /var/log/lab-maintenance.log 2>&1
  ```
- Reapply any custom `/etc/network/interfaces` tweaks if you had them.

---

## 7) (Optional) NVIDIA driver for Plex later

```bash
apt install -y pve-headers-$(uname -r)
apt install -y nvidia-driver firmware-misc-nonfree
reboot
nvidia-smi   # should show your GPU
```

(Then pass the GPU into a Plex LXC and enable **Hardware Transcoding** in Plex.)

---

## 8) Verification checklist

- [ ] **Datacenter → Storage** shows `VM-Storage` (green).
- [ ] VMs start and disks map to `VM-Storage`.
- [ ] LXCs start and `rootfs` points to `subvol-<VMID>-disk-0`.
- [ ] Web UI responsive; graphs updating.
- [ ] (Optional) `nvidia-smi` shows GPU after driver install.
- [ ] Add a diary entry with any tweaks you made.

---

## Troubleshooting tips

- **Wrong storage ID in configs:** change to `VM-Storage` or re-add storage under the old ID you used originally.
- **Cannot mount 1 TB disk:** check filesystem type (`lsblk -f`) and use matching fstab options (e.g., `xfs` vs `ext4`).
- **Guests won’t boot:** look at **Task Log** in the GUI, confirm disk paths exist, `qm config <VMID>` / `pct config <CTID>`.

---

## TL;DR (sticky note)

1) **Back up** `/etc/pve/{qemu-server,lxc}`, `storage.cfg`, `interfaces` → tar off-box.  
2) **Reinstall** PVE on **NVMe only** (leave 1 TB alone).  
3) **Mount + add** 1 TB as `VM-Storage` (same storage ID).  
4) **Restore configs** → fix disk paths → **start guests**.  
5) (Optional) install **NVIDIA** → `nvidia-smi`.
