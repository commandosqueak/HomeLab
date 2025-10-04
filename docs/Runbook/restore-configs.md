# 🔄 Restore Proxmox Configs from Backup

This guide explains how to restore VM/LXC configuration files in Proxmox from a backup tarball.

---

## 1. Copy backup tarballs into /root
```bash
cp /mnt/pve/VM-Storage/pve-backups/pve-configs.tgz /root/
cp /mnt/pve/VM-Storage/pve-backups/host-scripts.tgz /root/
```

---

## 2. Extract into a temporary folder
```bash
mkdir -p /root/tmp
tar -xzf /root/pve-configs.tgz -C /root/tmp
```

---

## 3. Restore configs to correct locations
```bash
# VM configs
cp /root/tmp/pve-backup/qemu/*.conf /etc/pve/qemu-server/

# LXC configs
cp /root/tmp/pve-backup/lxc/*.conf /etc/pve/lxc/

# System configs
cp /root/tmp/pve-backup/sys/storage.cfg /etc/pve/storage.cfg
cp /root/tmp/pve-backup/sys/hosts /etc/hosts
cp /root/tmp/pve-backup/sys/interfaces /etc/network/interfaces
```

---

## 4. Restart Proxmox services
```bash
systemctl restart pvedaemon pve-cluster pveproxy
```

---

## 5. Verify configs
```bash
ls /etc/pve/qemu-server/
ls /etc/pve/lxc/
```
You should see your VM and LXC config files (e.g., `200.conf`, `220.conf`, `9000.conf`, `230.conf`, `233.conf`).

---

## 6. Test in Web UI
1. Log in at `https://<proxmox-ip>:8006`  
2. VMs/LXCs should appear under **Datacenter → Node**.  
3. Start them one by one and verify dashboards are accessible.

---

✅ Done. VMs/LXCs are restored from backup configs.
