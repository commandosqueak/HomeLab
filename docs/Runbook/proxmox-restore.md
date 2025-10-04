# Proxmox Restore After Clean Reinstall — Step‑by‑Step

**Goal:** You reinstalled Proxmox VE on the boot NVMe and left the 1 TB VM disk untouched. This guide puts everything back: storage, VM/CT configs, cron, certs, etc.

> Assumptions
> - Boot disk: 256 GB NVMe (fresh Proxmox 8.x “bookworm”).
> - VM disk: 1 TB SATA mounted at **/mnt/pve/VM-Storage** (directory storage).
> - You previously saved: `pve-configs.tgz`, optionally `host-scripts.tgz`, plus folders `cron*` and `etc-ssl` under `/mnt/pve/VM-Storage/pve-backups/`.

---

## 1) First boot sanity (host side)

1. Login on console or web at `https://<host>:8006`.
2. Update repos to Proxmox **no-subscription** + Debian **bookworm**, then fully update:
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

3. (Optional) Recreate your previous hostname/IP if you changed it during install.

---

## 2) Reattach the 1 TB VM disk (non‑destructive)

1. Identify the disk and filesystem:
   ```bash
   lsblk -f
   ```
   Note the correct device (e.g., `/dev/sda1`) and FS (ext4/xfs).

2. Mount it at the expected path:
   ```bash
   mkdir -p /mnt/pve/VM-Storage
   mount /dev/sdX1 /mnt/pve/VM-Storage   # replace sdX1 with your actual partition
   ```

3. Persist the mount in `/etc/fstab`:
   ```bash
   blkid /dev/sdX1
   nano /etc/fstab
   # Add a line like (ext4 example):
   # UUID=<YOUR-UUID>  /mnt/pve/VM-Storage  ext4  defaults  0  2
   mount -a
   ```

4. Add the storage back to Proxmox **with the SAME ID** as before (`VM-Storage`):
   - **GUI:** Datacenter → Storage → **Add → Directory**
     - ID: `VM-Storage`
     - Directory: `/mnt/pve/VM-Storage`
     - Content: **Disk image, Container, ISO, Snippets** (tick as needed)
   - **CLI:**
     ```bash
     pvesm add dir VM-Storage --path /mnt/pve/VM-Storage --content images,rootdir,iso,snippets
     ```

5. Verify Proxmox sees the storage (should be green):
   ```bash
   pvesm status
   ls /mnt/pve/VM-Storage/images || true
   ls /mnt/pve/VM-Storage | sed -n '1,50p'
   ```

---

## 3) Restore VM/CT configuration files

1. Bring your saved tarball(s) back into `/root` (they live on the VM disk already):
   ```bash
   cp /mnt/pve/VM-Storage/pve-backups/pve-configs.tgz /root/
   tar -xzf /root/pve-configs.tgz -C /
   # This recreates /root/pve-backup/{qemu,lxc,sys}
   ```

2. Copy configs into Proxmox config dirs:
   ```bash
   cp -a /root/pve-backup/qemu/*.conf /etc/pve/qemu-server/ 2>/dev/null || true
   cp -a /root/pve-backup/lxc/*.conf  /etc/pve/lxc/         2>/dev/null || true
   ```

3. (Optional) Restore network/storage reference files for your own notes:
   ```bash
   cp -a /root/pve-backup/sys/storage.cfg /etc/pve/ 2>/dev/null || true
   cp -a /root/pve-backup/sys/interfaces  /etc/network/ 2>/dev/null || true
   cp -a /root/pve-backup/sys/hosts       /etc/        2>/dev/null || true
   ```
   > If you overwrite `/etc/network/interfaces`, **reboot** or `systemctl restart networking` after checking it’s correct. If unsure, skip this and configure via the GUI.

4. Confirm Proxmox now lists your guests:
   ```bash
   qm list
   pct list
   ```

---

## 4) Fix VM disks or LXC rootfs paths (if needed)

If any VM/CT shows a missing disk in the UI task log:

- **VMs:** open `/etc/pve/qemu-server/<VMID>.conf` and ensure lines point to real files on storage:
  ```
  scsi0: VM-Storage:images/<VMID>/vm-<VMID>-disk-0.qcow2
  ```
  Update via CLI example:
  ```bash
  qm set 100 --scsi0 VM-Storage:images/100/vm-100-disk-0.qcow2
  ```

- **LXCs:** ensure `rootfs` points to an existing subvolume (common pattern):
  ```
  rootfs: VM-Storage:subvol-230-disk-0,size=XXG
  ```
  Update via CLI example:
  ```bash
  pct set 230 -rootfs VM-Storage:subvol-230-disk-0
  ```

> Tip: list what’s actually there:
> ```bash
> ls -lh /mnt/pve/VM-Storage/images/<VMID>/
> ls -lh /mnt/pve/VM-Storage/ | grep subvol- | sed -n '1,120p'
> ```

Start guests **one by one** and watch the task logs:
```bash
qm start 100
pct start 230
```

---

## 5) Restore cron jobs, certs, and scripts (if you backed them up)

Everything below is **optional** and only if you previously saved it.

1. **Cron:** (system-wide and user crontabs)
   ```bash
   # system-wide cron dirs
   cp -a /mnt/pve/VM-Storage/pve-backups/cron* /etc/ 2>/dev/null || true
   # root user crontab
   [ -f /mnt/pve/VM-Storage/pve-backups/crontab ] && crontab /mnt/pve/VM-Storage/pve-backups/crontab
   ```

2. **TLS certs/keys:** (e.g., ACME/Let’s Encrypt under /etc/ssl)
   ```bash
   cp -a /mnt/pve/VM-Storage/pve-backups/etc-ssl /etc/ssl-backup-restore
   # Review what you need and copy selectively to /etc/ssl if appropriate
   # Example (SELF-SIGNED USERS CAN SKIP):
   # cp -a /etc/ssl-backup-restore/* /etc/ssl/
   systemctl restart pveproxy || true
   ```

3. **Host scripts:** (if you created `host-scripts.tgz`)
   ```bash
   cp /mnt/pve/VM-Storage/pve-backups/host-scripts.tgz /root/ 2>/dev/null || true
   tar -xzf /root/host-scripts.tgz -C /
   chmod +x /usr/local/sbin/*.sh 2>/dev/null || true
   ```

4. **Recreate any cron schedule** you used for maintenance:
   ```bash
   crontab -e
   # Example weekly:
   # 15 3 * * 0 /usr/local/sbin/lab-maintenance.sh >> /var/log/lab-maintenance.log 2>&1
   ```

---

## 6) Final checks

- **Storage:** `pvesm status` shows `VM-Storage` OK (green).
- **Guests:** `qm list` / `pct list` populated; each guest starts without errors.
- **Networking:** Guests get the expected IPs (or static as configured).
- **Backups:** Set a backup target (when you add PBS later) and run a manual test backup.

Optional: re‑enable the Proxmox **QEMU guest agent** per VM (Options → “QEMU Guest Agent” → Enable).

---

## 7) (Optional) NVIDIA for Plex — after you’re stable

```bash
apt install -y pve-headers-$(uname -r)
apt install -y nvidia-driver firmware-misc-nonfree
reboot
nvidia-smi
```
Then pass the GPU to your Plex LXC and enable **hardware transcoding** in Plex.

---

## 8) Troubleshooting quickies

- **VM not listed but disk exists:** create a minimal VM with the same VMID, attach the existing disk:
  ```bash
  qm create 100 --name rescued --memory 4096 --cores 2 --net0 virtio,bridge=vmbr0
  qm set 100 --scsi0 VM-Storage:images/100/vm-100-disk-0.qcow2
  qm set 100 --boot order=scsi0
  ```
- **LXC rootfs missing:** point `rootfs` to the correct `subvol-<CTID>-disk-0` then start.
- **Wrong device name after reinstall:** disks can reorder; always check `lsblk -f` and fix `/etc/fstab` accordingly.
- **pveproxy cert warning:** expected unless you restore custom certs; self-signed is fine for lab use.

---

## 9) Post-restore housekeeping

- Snapshot each guest now that it’s booting.
- Commit “Restore successful” notes to your runbook & diary.
- Plan PBS (Proxmox Backup Server) and a regular backup schedule.

---

**You’re done.** Your lab is back. Breathe, sip coffee, pet Ruby. 🐱
