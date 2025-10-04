# 🛠️ fstab Recovery Cheat Sheet

If Proxmox fails to boot because of a bad `/etc/fstab` entry (e.g., wrong UUID or FS type), follow these steps:

---

## 1. Detect the problem
- System hangs during boot showing `A start job is running for /mnt/...`
- Or Proxmox won’t finish booting at all.

---

## 2. Boot into Recovery
1. Reboot the server.  
2. At the GRUB menu (Proxmox boot screen), press `e` to edit the boot entry.  
3. Find the line starting with `linux` and append:
   ```
   systemd.unit=rescue.target
   ```
4. Press `Ctrl+X` (or `F10`) to boot into **Rescue mode**.

---

## 3. Remount root writable
By default, rescue mode mounts root as read-only. Make it writable:
```bash
mount -o remount,rw /
```

---

## 4. Edit fstab
Open the file:
```bash
nano /etc/fstab
```

Comment out (add `#` at the start) the line causing the issue.  
Example:
```
# UUID=d895a34f-621c-41f7-97f6-a1202124fbf0 /mnt/pve/VM-Storage ext4 defaults 0 2
```

Save (`Ctrl+O`, `Enter`) and exit (`Ctrl+X`).

---

## 5. Reboot
```bash
reboot
```

Proxmox should boot normally again.  
You can then re-check the correct UUID/filesystem with:
```bash
lsblk -f
blkid
```
…and fix `/etc/fstab` properly.

---

✅ That’s it. Safe and repeatable every time.
