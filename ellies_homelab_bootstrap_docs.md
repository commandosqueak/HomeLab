# Ellie's Homelab Bootstrap — Starter Repo

Welcome! This doc is a practical, **copy‑pasteable** starter for your homelab: OPNsense baseline automation, Ansible snippets, docker-compose stubs, DDNS scripts, backup guidance, and the bare‑metal OPNsense fallback plan. All secrets are placeholders — **replace** them before running anything.

---

## Repo layout (suggested)
```
homelab/
├─ opnsense/
│  ├─ config-sanitized.xml       # export from OPNsense with placeholders
│  └─ opnsense-bootstrap.sh      # API bootstrap script
├─ ansible/
│  ├─ roles/role-wallpaper/
│  └─ playbooks/set-wallpaper.yml
├─ compose/
│  ├─ pihole-docker-compose.yml
│  ├─ languagetool-compose.yml
│  └─ utils-compose.yml         # portainer, uptime-kuma, wireguard (if desired)
├─ ddns/
│  ├─ duckdns-update.sh
│  └─ cloudflare-update.sh
├─ backups/
│  └─ pbs-rclone-sync.sh
├─ docs/
│  └─ network-diagrams.png
└─ README.md
```

---

## 1) OPNsense baseline strategy

Two recommended approaches (both are safe to use together):

**A — Sanitized `config.xml` restore (fast):**
1. Build OPNsense manually once.
2. Configure VLANs, firewall base rules, VPN placeholders, MaxMind plugin, admin user & 2FA, SSH key auth.
3. Export **System → Configuration → Backups** as `config-sanitized.xml`.
4. Replace secrets with placeholders (e.g. `YOUR_MAXMIND_KEY`, `WG_PUBKEY`, `API_KEY`).
5. Check into `opnsense/config-sanitized.xml` in your repo (private!).

**B — API-driven deltas (idempotent):**
- Use the included `opnsense-bootstrap.sh` to restore and apply small plugin/rule changes. Good for incremental automation.

> _Tip_: keep `config-sanitized.xml` private. Use git private repo or local-only storage.

### `opnsense-bootstrap.sh` (example)
```bash
#!/usr/bin/env bash
# opnsense-bootstrap.sh
HOST="https://OPNSENSE_HOST_OR_IP"
APIKEY="OPNSENSE_API_KEY"
APISECRET="OPNSENSE_API_SECRET"

# 1) Restore config (local file path) - only if you intend a full restore
curl -sk -u "$APIKEY:$APISECRET" \
  -F "conffile=@config-sanitized.xml" \
  "$HOST/diag_backup.php?restore=1"

# 2) Install plugins (example: MaxMind)
curl -sk -u "$APIKEY:$APISECRET" -H "Content-Type: application/json" \
  -d '{"plugins":["os-maxmind"]}' \
  "$HOST/api/core/firmware/installPlugin"

# 3) Create a GeoIP alias (example)
curl -sk -u "$APIKEY:$APISECRET" -H "Content-Type: application/json" \
  -d '{"alias":{"name":"BLOCK_COUNTRIES","type":"geoip","content":["CN","RU","KP"],"descr":"GeoIP block"}}' \
  "$HOST/api/firewall/alias/addItem"

# 4) Reload rules
curl -sk -u "$APIKEY:$APISECRET" "$HOST/api/firewall/filter/reload"

echo "OPNsense bootstrap: done (check GUI to fill in any secret placeholders)."
```

Notes:
- API endpoints/names may change between OPNsense versions. If an endpoint fails, check `<OPNsenseURL>/api/` or the GUI API docs.
- Always test on a clone or lab instance first.

---

## 2) Bare‑metal OPNsense fallback plan
Keep two separate drives:
- Drive A: Proxmox (primary boot)
- Drive B: OPNsense bare‑metal install (golden fallback)

Workflow:
1. Install Proxmox to Drive A. Install OPNsense VM(s) on Proxmox, configured identically to your bare‑metal plan.
2. Install OPNsense bare‑metal to Drive B using the same `config-sanitized.xml` (or export/import to ensure parity).
3. BIOS default boot = Drive A. If Proxmox host fails, change BIOS to boot Drive B and you’re back online.

Important: do not attempt to share a single disk for both roles.

---

## 3) Docker compose stubs (drop into `compose/`)

### Pi-hole (`pihole-docker-compose.yml`)
```yaml
version: '3'
services:
  pihole:
    image: pihole/pihole:latest
    container_name: pihole
    environment:
      TZ: 'Europe/London'
      WEBPASSWORD: 'CHANGE_ME'
      DNSMASQ_USER: 'pihole'
    volumes:
      - ./etc-pihole:/etc/pihole
      - ./etc-dnsmasq.d:/etc/dnsmasq.d
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "80:80"
    cap_add:
      - NET_ADMIN
    restart: unless-stopped
```

### LanguageTool (`languagetool-compose.yml`)
```yaml
version: '3'
services:
  languagetool:
    image: silviof/docker-languagetool:latest
    container_name: languagetool
    ports:
      - "8010:8010"
    environment:
      - Java_Xms=256m
      - Java_Xmx=2048m
    restart: unless-stopped
```

### Portainer + Uptime Kuma + (optional) WireGuard (`utils-compose.yml`)
```yaml
version: '3'
services:
  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    ports:
      - "9443:9443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./portainer:/data
    restart: unless-stopped

  uptime-kuma:
    image: louislam/uptime-kuma:latest
    container_name: uptime-kuma
    ports:
      - "3001:3001"
    volumes:
      - ./uptime-kuma:/app/data
    restart: unless-stopped

  # Optional: wireguard if you want containerized WG as an alternative
  wireguard:
    image: linuxserver/wireguard
    container_name: wireguard
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - SERVERURL=YOUR_DDNS_HOSTNAME
      - SERVERPORT=51820
    volumes:
      - ./wg-config:/config
      - /lib/modules:/lib/modules
    ports:
      - "51820:51820/udp"
    restart: unless-stopped
```

---

## 4) Wallpaper automation (Ansible role snippet)
`ansible/roles/role-wallpaper/tasks/main.yml`
```yaml
- name: Copy role wallpaper
  copy:
    src: "files/{{ wallpaper }}"
    dest: /usr/share/backgrounds/role.png
    mode: '0644'

- name: Set GNOME background (example)
  become: yes
  shell: |
    sudo -u {{ ansible_user_id }} dbus-launch gsettings set org.gnome.desktop.background picture-uri 'file:///usr/share/backgrounds/role.png'
  args:
    executable: /bin/bash
  changed_when: false
```

Playbook example `ansible/playbooks/set-wallpaper.yml` binds hostnames to wallpapers.

---

## 5) Free DDNS examples

### DuckDNS (`ddns/duckdns-update.sh`)
```bash
#!/usr/bin/env bash
DOMAIN=yourname
TOKEN=YOUR_DUCKDNS_TOKEN
curl -s "https://www.duckdns.org/update?domains=$DOMAIN&token=$TOKEN&ip="
```
Cron example: `*/5 * * * * /path/to/duckdns-update.sh >/dev/null 2>&1`

### Cloudflare (if you own a domain) (`ddns/cloudflare-update.sh`)
```bash
#!/usr/bin/env bash
ZONE_ID=YOUR_ZONE_ID
RECORD_ID=YOUR_RECORD_ID
TOKEN=YOUR_API_TOKEN
IP=$(curl -s ifconfig.me)
curl -sX PUT "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/dns_records/$RECORD_ID" \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  --data "{\"type\":\"A\",\"name\":\"home\",\"content\":\"$IP\",\"ttl\":120,\"proxied\":false}"
```

---

## 6) Backups: quick plan
- **Proxmox → Proxmox Backup Server (PBS)** on NAS or a dedicated VM.
  - Incremental & deduped saves.
- **PBS/VM snapshots → Cloud** using `rclone` to Backblaze B2 / Wasabi / S3.
- **Veeam** if you prefer agent-based app-consistent backups for Windows workloads.

Example `backups/pbs-rclone-sync.sh` (very basic):
```bash
#!/usr/bin/env bash
# sync PBS datastore to rclone remote (rclone configured separately)
PBS_REPO=your-pbs-repo
RCLONE_REMOTE=b2:homelab-backups/pbs
rclone copy /var/lib/proxmox-backup/$PBS_REPO $RCLONE_REMOTE --transfers=4 --check-first
```

---

## 7) LLM + LanguageTool hosting notes
- Host LLM on **Main LAN** with a static IP (e.g. `10.20.10.20`).
- Firewall: allow Main LAN -> LLM only.
- Remote access only via VPN-in (WireGuard/OpenVPN) to prevent exposing model to WAN.
- If using GPU/TensorRT, ensure your server node has proper drivers and Docker runtime (nvidia-docker) configured.

---

## 8) VPN double-tunnel checklist
- Set outbound VPN client in OPNsense; mark as a gateway.
- Set inbound WireGuard/OpenVPN server on OPNsense for remote clients.
- Create firewall policies so remote clients route through the outbound VPN gateway.
- Test DNS leak protection (force DNS queries through Pi-hole and firewall rules).

---

## 9) IP & VLAN cheat-sheet (your 10.20.x.x plan)
- VLAN 10: `10.20.10.0/24` — Main LAN — GW `10.20.10.1` (static LAN IPs)
- VLAN 20: `10.20.20.0/24` — IoT — GW `10.20.20.1` (DHCP on WLAN for IoT)
- VLAN 30: `10.20.30.0/24` — Guest — GW `10.20.30.1` (DHCP)
- VLAN 40: `10.20.40.0/24` — Lab — GW `10.20.40.1`
- VLAN 50: `10.20.50.0/24` — DMZ — GW `10.20.50.1` (optional)

Example: Static hosts section in OPNsense for key services (LLM, NAS, Pi-hole, HA):
- `10.20.10.10` — Proxmox management
- `10.20.10.20` — LLM
- `10.20.10.30` — NAS management
- `10.20.10.40` — Home Assistant (if static)
- `10.20.10.53` — Pi-hole (if you want DNS to be on .53 locally)

---

## 10) Documentation strategy (recommended)
- **Git** (private repo): store all machine-parseable items — `config.xml`, `docker-compose`, scripts, Ansible. Version control + PRs.
- **Obsidian**: store runbooks, how-tos, troubleshooting notes, diagrams, recovery steps. Great for human-readable procedural docs.
- **phpIPAM** (optional): authoritative IP address management once you scale beyond 50 devices. Use it for static reservations, rack layout, etc.

Workflow: edit configs in Git, write runbook in Obsidian that references the Git files. Keep backups of the Git repo in your NAS + cloud.

---

## 11) Quick security hygiene
- Use strong passwords + 2FA where possible (OPNsense, Portainer, NAS, cloud accounts).
- Keep OPNsense updated (you preferred frequent patches — OPNsense makes that easy).
- Use separate admin accounts per service. Avoid reusing passwords.

---

## 12) Next steps & checklist
- [ ] Create private Git repo and add this doc and subfolders
- [ ] Stand up OPNsense in Proxmox and harden once; export sanitized `config.xml`
- [ ] Install MaxMind plugin and configure GeoIP alias
- [ ] Deploy Pi-hole, Portainer, Uptime-Kuma, LanguageTool containers
- [ ] Configure WireGuard (VPN-in) + provider VPN client (VPN-out) in OPNsense
- [ ] Test remote double-tunnel from phone/laptop
- [ ] Build bare-metal OPNsense drive and verify BIOS boot flip
- [ ] Configure backups (PBS + rclone to cloud)

---

If you want, I can now:
- generate the repo files (sanitized placeholders) and drop them into the canvas for you to copy, or
- produce a downloadable ZIP of all files.

Tell me which and I’ll create the files next.

