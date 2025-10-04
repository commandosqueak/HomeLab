# Weekly Lab Maintenance — Snapshots → Update → Health → Prune

**Goal:** create safety-point snapshots, update everything (containers + selected Debian guests), verify health, then prune last week’s snapshots so only the newest cycle remains.  
**Scope:** All VMs/CTs you list (except Proxmox host), Docker stacks on the Apps VM.  
**Retention:** Keep **1** weekly cycle (i.e., last week only).

## Inventory (adjust to match your IDs/IPs)
- **VMs**
  - `100` — Apps VM (Docker)
  - `220` — Home Assistant
  - *(add OPNsense VM ID later if you want snapshot-only before manual upgrades)*

- **CTs**
  - `230` — Pi-hole
  - `233` — WireGuard (+ WGDashboard)

- **Docker stacks (on Apps VM @ 10.20.10.25)**
  - Vaultwarden — `/opt/stacks/vaultwarden`
  - UniFi Controller — `/opt/stacks/unifi`
  - Glances — `/opt/stacks/glances`
  - Dozzle — `/opt/stacks/dozzle`
  - Portainer — `/opt/stacks/portainer` *(optional—update manually if you prefer)*
  - Linkding — `/opt/stacks/linkding`
  - Homebox — `/opt/stacks/homebox`
  - Uptime Kuma — `/opt/stacks/uptime-kuma`
  - LanguageTool — `/opt/stacks/languagetool`
  - DuckDNS — `/opt/stacks/duckdns`
  - Heimdall — `/opt/stacks/heimdall`
  - Plex — `/opt/stacks/plex` *(prepared; start later when NAS is up)*

- **Health checks (HTTP 200 expected)**
  - `http://10.20.10.25:8085` (Vaultwarden)
  - `https://10.20.10.25:8443` (UniFi — self-signed; we allow insecure)
  - `http://10.20.10.25:61208` (Glances)
  - `http://10.20.10.25:3001` (Uptime Kuma)
  - *(add more once you confirm ports for Linkding/Homebox/Heimdall/Plex, etc.)*

---

## Script: `/usr/local/sbin/lab-maintenance.sh`

> Runs on the **Proxmox host**. Snapshots VMs/CTs, updates Debian-based guests & Docker stacks on the Apps VM via SSH, performs health checks, then prunes old snapshots. Leaves the Proxmox **host** itself alone (you’ll update that manually).

```bash
#!/usr/bin/env bash
set -euo pipefail

# ====== USER CONFIG ======
VMS=(100 220)            # VM IDs: 100=Apps, 220=HomeAssistant
CTS=(230 233)            # CT IDs: 230=Pi-hole, 233=WireGuard

KEEP=1                   # keep last 1 cycle (1 week)
SNAP_PREFIX="auto-weekly"

# Debian/Ubuntu apt upgrades inside guests
APT_UPGRADE_CTS=(230 233)
APT_UPGRADE_VMS_SSH=("ellie@10.20.10.25")    # Apps VM SSH

# Docker stacks on Apps VM
APPS_SSH="ellie@10.20.10.25"
DOCKER_STACK_DIRS=(
  "/opt/stacks/vaultwarden"
  "/opt/stacks/unifi"
  "/opt/stacks/glances"
  "/opt/stacks/dozzle"
  "/opt/stacks/plex"
  "/opt/stacks/portainer"
  "/opt/stacks/linkding"
  "/opt/stacks/homebox"
  "/opt/stacks/uptime-kuma"
  "/opt/stacks/languagetool"
  "/opt/stacks/duckdns"
  "/opt/stacks/heimdall"
)

# Health URLs (expect 200 OK; UniFi is self-signed so we curl -k)
HEALTH_URLS=(
  "http://10.20.10.25:8085"
  "https://10.20.10.25:8443"
  "http://10.20.10.25:61208"
  "http://10.20.10.25:3001"
)

MAX_LOAD_AVG=6.0

# ====== FUNCTIONS ======
ts(){ date +"%Y-%m-%d %H:%M:%S"; }
log(){ echo "[$(ts)] $*"; }

check_load(){
  local la; la=$(awk '{print $1}' /proc/loadavg)
  awk -v a="$la" -v m="$MAX_LOAD_AVG" 'BEGIN{exit !(a<m)}' || { log "Load $la >= $MAX_LOAD_AVG; abort."; exit 1; }
}

snapshot_vm(){ qm snapshot "$1" "$2" --description "auto weekly pre-update" --vmstate 0; }
snapshot_ct(){ pct snapshot "$1" "$2" --description "auto weekly pre-update"; }

list_snap_vm(){ qm listsnapshot "$1" | awk -v p="$2" '$0 ~ p {print $1}' | sort; }
list_snap_ct(){
  if pct listsnapshot "$1" &>/dev/null; then pct listsnapshot "$1" | awk -v p="$2" '$0 ~ p {print $1}' | sort; else echo ""; fi
}

prune_vm(){
  local snaps; snaps=$(list_snap_vm "$1" "$3"); local n; n=$(echo "$snaps" | sed '/^$/d' | wc -l)
  local del=$((n - $2)); (( del>0 )) && for s in $(echo "$snaps" | head -n "$del"); do qm delsnapshot "$1" "$s"; done
}
prune_ct(){
  local snaps; snaps=$(list_snap_ct "$1" "$3"); [[ -z "$snaps" ]] && return
  local n; n=$(echo "$snaps" | sed '/^$/d' | wc -l)
  local del=$((n - $2)); (( del>0 )) && for s in $(echo "$snaps" | head -n "$del"); do pct delsnapshot "$1" "$s"; done
}

apt_upgrade_ct(){ pct exec "$1" -- bash -lc 'export DEBIAN_FRONTEND=noninteractive; apt-get update && apt-get -y dist-upgrade && apt-get -y autoremove --purge && apt-get -y autoclean'; }
apt_upgrade_vm_ssh(){
  ssh -o BatchMode=yes -o StrictHostKeyChecking=accept-new "$1"   'export DEBIAN_FRONTEND=noninteractive; sudo apt-get update && sudo apt-get -y dist-upgrade && sudo apt-get -y autoremove --purge && sudo apt-get -y autoclean'
}

update_docker_stacks(){
  ssh -o BatchMode=yes -o StrictHostKeyChecking=accept-new "$APPS_SSH" bash -s <<'EOSSH'
set -euo pipefail
DIRS=(
  "/opt/stacks/vaultwarden" "/opt/stacks/unifi" "/opt/stacks/glances" "/opt/stacks/dozzle"
  "/opt/stacks/plex" "/opt/stacks/portainer" "/opt/stacks/linkding" "/opt/stacks/homebox"
  "/opt/stacks/uptime-kuma" "/opt/stacks/languagetool" "/opt/stacks/duckdns" "/opt/stacks/heimdall"
)
for d in "${DIRS[@]}"; do
  if [ -f "$d/docker-compose.yml" ] || [ -f "$d/compose.yml" ]; then
    echo "[Docker] $d: pull + up -d"
    cd "$d"; docker compose pull; docker compose up -d
  fi
done
docker image prune -f
EOSSH
}

health_checks(){
  local ok=1
  for url in "${HEALTH_URLS[@]}"; do
    if curl -k -sS -m 10 -o /dev/null -w "%{http_code}" "$url" | grep -q '^2'; then
      log "[HEALTH] OK: $url"
    else
      log "[HEALTH] FAIL: $url"; ok=0
    fi
  done
  return $ok
}

# ====== MAIN ======
check_load
LABEL="${SNAP_PREFIX}-$(date +%Y%m%d)"
log "=== SNAPSHOTS: $LABEL ==="
for id in "${VMS[@]}"; do log "VM $id"; snapshot_vm "$id" "$LABEL"; done
for id in "${CTS[@]}"; do log "CT $id"; snapshot_ct "$id" "$LABEL"; done

log "=== APT UPGRADES ==="
for id in "${APT_UPGRADE_CTS[@]}"; do log "CT $id apt"; apt_upgrade_ct "$id"; done
for t in "${APT_UPGRADE_VMS_SSH[@]}"; do log "VM $t apt"; apt_upgrade_vm_ssh "$t"; done

log "=== DOCKER STACK UPDATES ==="
update_docker_stacks

log "=== HEALTH CHECKS ==="
if ! health_checks; then
  log "Health checks failed — leaving snapshots in place."; exit 2
fi

log "=== PRUNE OLD SNAPSHOTS (keep $KEEP) ==="
for id in "${VMS[@]}"; do prune_vm "$id" "$KEEP" "${SNAP_PREFIX}-"; done
for id in "${CTS[@]}"; do prune_ct "$id" "$KEEP" "${SNAP_PREFIX}-"; done

log "Done."
```

---

## Runbook Notes
- **Storage:** Snapshots require ZFS or LVM-thin. Move any guest disks to snapshot-capable storage.
- **SSH:** Proxmox host needs SSH key auth to the Apps VM (`ellie@10.20.10.25`) for updates.
- **Host updates:** Proxmox VE host is *not* touched by this script (update it manually).
- **OPNsense:** Once online, add its VMID to `VMS=(...)` for **snapshot-only** before manual upgrades.
- **PBS later:** When PBS is running, schedule PBS backups **before** this script. This script remains the “second safety net.”
