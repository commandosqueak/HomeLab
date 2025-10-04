# Weekly Maintenance Checklist (Snapshots → Update → Health → Prune)

**When:** Sundays @ 03:15 (cron) — or run manually before big changes  
**Goal:** Create safety snapshots → update guests & containers → verify → prune old snapshots

## 0) Pre-flight
- [ ] PBS backups from last night are **green** (when PBS is online)
- [ ] Proxmox host load is low (`uptime`)
- [ ] Disk space OK on pools/vols (`zpool list`, `df -h`)
- [ ] Quick glance at Uptime Kuma (no major reds)

## 1) Snapshots (Proxmox host)
- [ ] Run `lab-maintenance.sh` **or** confirm cron executed last run
- [ ] Verify snapshots exist for all in-scope guests:
  - VMs: `100 Apps`, `220 HomeAssistant` *(add OPNsense VMID later for snap-only)*
  - CTs: `230 Pi-hole`, `233 WireGuard`
- [ ] Snapshot label format: `auto-weekly-YYYYMMDD`

## 2) Guest updates
- [ ] LXC apt updates (inside CTs): Pi-hole (230), WireGuard (233)
- [ ] VM apt updates via SSH (Apps VM 100 @ `10.20.10.25`)
- [ ] **Do not** auto-update the Proxmox **host** here (manual only)

## 3) Docker stacks (Apps VM)
- [ ] `docker compose pull && up -d` for stacks:
  - Vaultwarden, UniFi, Glances, Dozzle, Portainer, Linkding, Homebox,
    Uptime-Kuma, LanguageTool, DuckDNS, Heimdall, *(Plex later when NAS ready)*
- [ ] `docker image prune -f` (cleanup)

## 4) Health checks
- [ ] `http://10.20.10.25:8085` Vaultwarden → **200 OK**
- [ ] `https://10.20.10.25:8443` UniFi UI → loads (self-signed)
- [ ] `http://10.20.10.25:61208` Glances → loads
- [ ] `http://10.20.10.25:3001` Uptime Kuma → green
- [ ] Add others (Heimdall/Linkding/Homebox/Plex) as you bring them online

> If any health check fails: **stop here** (do not prune). Investigate or roll back to the fresh snapshot.

## 5) Prune old snapshots
- [ ] Keep only **1** weekly cycle (latest). Confirm old `auto-weekly-*` removed.

## 6) Notes & diary
- [ ] Record run result in **Diary/YYYY-MM-DD** (what updated, any issues)
- [ ] Open issues added to **To-Do** with priority

## Rollback quick steps (if needed)
- Proxmox GUI → VM/CT → **Snapshots** → select `auto-weekly-YYYYMMDD` → **Rollback**
- For container-only snafus: `docker compose down`, pin previous image tag, or restore a VM snapshot

## Safety reminders
- Don’t auto-update **OPNsense** or **Proxmox host** in this workflow
- Ensure VM disks sit on **ZFS** or **LVM-thin** (snapshot-capable)
- Consider a manual snapshot before major app version jumps
- Once PBS is live, let **PBS backups** run **before** this checklist

## Nice-to-haves
- [ ] QEMU guest agent installed & enabled in VMs
- [ ] NTP synced on all hosts/VMs
- [ ] UPS/NUT status OK
- [ ] Spare known-good kernel kept on Proxmox host for a week after upgrades

(And give Ruby a treat after a clean run 😺)
