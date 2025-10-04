# Install & Schedule: Weekly Maintenance Script

## 1) Save the script
```bash
sudo nano /usr/local/sbin/lab-maintenance.sh
# (paste the script from maintenance.md)
sudo chmod 750 /usr/local/sbin/lab-maintenance.sh
```

## 2) SSH trust from Proxmox → Apps VM
```bash
# On Proxmox host:
ssh-keygen -t ed25519 -N "" -f ~/.ssh/id_ed25519
ssh-copy-id ellie@10.20.10.25
# Test:
ssh ellie@10.20.10.25 'echo ok'
```

## 3) Verify snapshot capability
- Ensure all VM/CT disks are on **ZFS** or **LVM-thin** storage.
- Install **qemu-guest-agent** in VMs (optional but recommended).

## 4) Dry run (manual)
```bash
sudo /usr/local/sbin/lab-maintenance.sh
# Watch output; verify snapshots created, docker stacks updated, health checks pass.
```

## 5) Schedule weekly (Sun 03:15)
```bash
sudo crontab -e
# add:
15 3 * * 0 /usr/local/sbin/lab-maintenance.sh >> /var/log/lab-maintenance.log 2>&1
```

## 6) Rollback (if needed)
- Proxmox GUI → VM/CT → **Snapshots** → select `auto-weekly-YYYYMMDD` → **Rollback**.
- For Docker-only issues, you can also `docker compose` pin previous image tags or restore a VM snapshot.

## 7) Customise
- **Add/remove** VM/CT IDs in the arrays.
- Edit `DOCKER_STACK_DIRS` to match what you actually run.
- Add more **HEALTH_URLS** (Heimdall, Linkding, Homebox, Plex when live).
- Adjust `KEEP` if you want more than one weekly cycle retained.
