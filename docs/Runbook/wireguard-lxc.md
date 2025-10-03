# WireGuard VPN — LXC Runbook (CT 233)

(Initial sections omitted in this export.)

---

## 9) WGDashboard Hardening (do once OPNsense & static IPs are ready)

> Keep the dashboard **LAN-only**. Never expose it to WAN. Perform these when your IP plan and OPNsense are live.

### A) Bind the dashboard to LAN IP only
Edit Gunicorn bind address:
```bash
sudo sed -i "s/^bind\s*=.*/bind = '192.168.8.184:10086'/" /etc/wgdashboard/src/gunicorn.conf.py
sudo systemctl restart wg-dashboard
```

### B) Proxmox CT firewall guard rails
- Proxmox → CT 233 → **Firewall: Enabled**
- Rules (Inbound):
  - **ACCEPT** src `192.168.8.0/24` dst port `10086/tcp`
  - **DROP**   any → dst port `10086/tcp`

### C) File permissions
```bash
sudo chmod 600 /etc/wireguard/wg0.conf /etc/wgdashboard/src/wg-dashboard.ini
sudo chown root:root /etc/wireguard/wg0.conf /etc/wgdashboard/src/wg-dashboard.ini
```

### D) OPNsense rules (when live)
- Port-forward **UDP 51820** → WG CT IP (192.168.8.184 now; 10.20.10.24 later).
- Create alias `WG_NET = 10.6.0.0/24`.
- Allow `WG_NET` only to the VLANs/services you want (least privilege).
- Optional: GeoIP allowlist on the WAN rule.

### E) Password gotchas
Avoid `#` and `;` in the WGDashboard password (INI treats them as comments).

