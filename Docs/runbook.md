# 🛠️ Homelab First Setup Runbook

## 1. Core Infrastructure
1. **Install Proxmox on Drive A (main drive)**
   - During setup:
     - Enable ZFS or ext4 (your preference).
     - Add static IP (e.g., `10.20.10.10`).
   - Update: `apt update && apt full-upgrade`.

2. **Install OPNsense bare-metal on Drive B (fallback)**
   - Use a USB installer, assign the same LAN IP (`10.20.10.1`).
   - Import sanitized `config-sanitized.xml`.
   - Test: unplug Drive A, boot from Drive B, make sure WAN→LAN works.

---

## 2. OPNsense VM (Primary Firewall/Router)
1. Create an OPNsense VM in Proxmox:
   - 2–4 vCPUs, 4–8 GB RAM.
   - Pass through 2 NICs (WAN + LAN).
2. Install OPNsense on the VM.
3. Import sanitized `config-sanitized.xml`.
4. Update packages, install `os-maxmind` plugin.
5. Configure VLAN interfaces:
   - VLAN 10 → Main LAN `10.20.10.1/24`.
   - VLAN 20 → IoT `10.20.20.1/24`.
   - VLAN 30 → Guest `10.20.30.1/24`.
   - VLAN 40 → Lab `10.20.40.1/24`.
6. Enable DHCP only on WLAN-facing networks (IoT/Guest).

---

## 3. Networking Hardware
1. **Switch setup**
   - Core switch uplink = SFP+ 10G to Proxmox.
   - Trunk ports: carry all VLANs to PoE switch.
   - Access ports: tag VLAN appropriately for APs.
2. **Access Points**
   - Configure SSIDs → map to VLANs:
     - SSID `Main` → VLAN 10.
     - SSID `IoT` → VLAN 20.
     - SSID `Guest` → VLAN 30.

---

## 4. Essential Services
1. **Pi-hole**
   - Deploy via `docker-compose`.
   - Assign static IP (e.g., `10.20.10.53`).
   - Point OPNsense DHCP/DNS at Pi-hole.

2. **Home Assistant**
   - Deploy on Proxmox VM or LXC.
   - Assign static IP (e.g., `10.20.10.40`).

3. **LLM server**
   - Deploy in Proxmox VM/Docker host.
   - Assign static IP (e.g., `10.20.10.20`).

---

## 5. Utility Stack
1. **Portainer** — deploy via compose, manage all containers.
2. **Uptime Kuma** — monitor uptime for NAS, APs, services.
3. **LanguageTool** — deploy via compose for grammar assistance.

---

## 6. VPN Setup
1. **Outbound VPN (to provider)**
   - OPNsense → VPN client → provider.
   - Mark VPN gateway as default.

2. **Inbound VPN (WireGuard/OpenVPN)**
   - OPNsense → VPN server.
   - Configure clients on phone/laptop.
   - Policy: inbound VPN clients route out via outbound VPN.

---

## 7. Backups
1. Install **Proxmox Backup Server** (on NAS or VM).
2. Schedule VM backups nightly → PBS datastore.
3. Use **rclone** to sync PBS datastore to Backblaze/Wasabi.
4. For Windows workloads, set up **Veeam Agent** → NAS.

---

## 8. Documentation
- Commit configs/scripts to **Git repo**.
- Write procedures, troubleshooting, and notes in **Obsidian**.
- Optional: manage IPs in **phpIPAM**.

---

## 9. Finishing Touches
- Ansible wallpaper role → set custom background per machine.
- Test fallback: reboot from Drive B (bare-metal OPNsense).
- Test VPN double tunnel (remote → OPNsense → provider).
- Test power outage recovery with UPS.

---

## Chapter 1: Proxmox Installation & Basic Setup

### 1.1. Prepare Hardware
- Insert bootable USB with Proxmox ISO (download from [proxmox.com](https://www.proxmox.com/en/downloads)).
- Ensure you have:
  - Drive A (main Proxmox install).
  - Drive B (reserved for bare-metal OPNsense fallback).
  - At least 2 NICs (WAN + LAN).

### 1.2. Install Proxmox
1. Boot from Proxmox USB.
2. Select **Install Proxmox VE**.
3. Storage:
   - Install on **Drive A** (your chosen Proxmox disk).
   - Filesystem: ZFS (for redundancy) or ext4 (simpler).
4. Region/keyboard → set to your locale.
5. Password + email → record in documentation.
6. Management network:
   - Assign static IP (e.g., `10.20.10.10`).
   - Gateway: your ISP modem/router (temporary until OPNsense is up).
   - DNS: `1.1.1.1` or temporary.

### 1.3. Post-Install Setup
1. Boot into Proxmox.
2. Login at: `https://10.20.10.10:8006`.
3. Run updates:
   ```bash
   apt update && apt full-upgrade -y
   ```
4. (Optional) Add Proxmox community repo:
   ```bash
   nano /etc/apt/sources.list
   ```
   Replace enterprise line with:
   ```
   deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription
   ```
5. Restart Proxmox.

---

## Chapter 2: OPNsense Installation & Basic Setup

### 2.1. Bare-Metal Fallback Install (Drive B)
1. Boot from OPNsense USB.
2. Install to **Drive B**.
3. Assign interfaces:
   - WAN = NIC1.
   - LAN = NIC2.
4. LAN IP: `10.20.10.1/24`.
5. Web UI: `https://10.20.10.1` (from a laptop on LAN).
6. Run setup wizard:
   - Hostname: `opnsense.local`.
   - Domain: `home.lab`.
   - DNS: Pi-hole (later), for now `1.1.1.1`.
   - WAN: DHCP (temporary).
   - LAN: Static `10.20.10.1`.

*(This ensures you can always boot straight into OPNsense if Proxmox dies.)*

---

### 2.2. OPNsense VM (Primary Router)
1. In Proxmox → create new VM:
   - Name: `opnsense-vm`.
   - Storage: 20–40GB.
   - CPU: 2–4 cores.
   - RAM: 4–8GB.
2. Upload OPNsense ISO to Proxmox local storage.
3. Add **2 NICs** (pass through from host):
   - `vmbr0` = WAN.
   - `vmbr1` = LAN.
4. Boot and install OPNsense.
5. Assign interfaces:
   - WAN (DHCP or static public IP).
   - LAN `10.20.10.1/24`.

---

### 2.3. Basic OPNsense Setup
1. Access web UI: `https://10.20.10.1`.
2. Change admin password immediately.
3. Update system:  
   **System → Firmware → Updates → Check for updates**.
4. Install GeoIP plugin:
   - **System → Firmware → Plugins** → install `os-maxmind`.
5. Configure VLANs:
   - VLAN 10 → Main LAN `10.20.10.1/24`.
   - VLAN 20 → IoT `10.20.20.1/24`.
   - VLAN 30 → Guest `10.20.30.1/24`.
   - VLAN 40 → Lab `10.20.40.1/24`.
6. Setup DHCP:
   - Only for WLAN VLANs (IoT + Guest).
   - LAN VLANs (wired) will remain static.
## Chapter 3: DNS Stack – Pi-hole & Dynamic DNS

### 3.1. Pi-hole Deployment (Docker)

1. On your Proxmox host, create a lightweight VM or LXC container for Pi-hole.
   - Recommended: 1 vCPU, 512MB RAM, 5GB storage.
   - Static IP: `10.20.10.53`.

2. Install Docker + Docker Compose:
   ```bash
   apt update && apt install -y docker.io docker-compose
   ```

3. Create `docker-compose.yml` for Pi-hole:
   ```yaml
   version: '3'
   services:
     pihole:
       container_name: pihole
       image: pihole/pihole:latest
       environment:
         TZ: 'UTC'
         WEBPASSWORD: 'changeme'
       volumes:
         - './etc-pihole:/etc/pihole'
         - './etc-dnsmasq.d:/etc/dnsmasq.d'
       ports:
         - '53:53/tcp'
         - '53:53/udp'
         - '80:80/tcp'
       restart: unless-stopped
   ```

4. Start container:
   ```bash
   docker-compose up -d
   ```

5. Access web UI:
   - URL: `http://10.20.10.53/admin`
   - Login with password you set above.

---

### 3.2. Integration with OPNsense

1. In **OPNsense Web UI**:
   - Go to **System → Settings → General**.
   - Set DNS server to `10.20.10.53` (Pi-hole IP).
   - Disable “Allow DNS server list to be overridden by DHCP/PPP.”

2. In **Services → DHCPv4 → [LAN/VLAN]**:
   - Set DNS server option to `10.20.10.53`.

3. Save and apply.  
   *Now all LAN and VLAN devices will resolve through Pi-hole.*

---

### 3.3. Pi-hole VLAN Awareness (Optional)

If you want per-VLAN filtering:
- Enable conditional forwarding in Pi-hole to OPNsense.
- Or run separate Pi-hole instances for critical VLANs.

---

### 3.4. Dynamic DNS (DuckDNS)

1. Create an account at [duckdns.org](https://www.duckdns.org/).
   - Add a subdomain, e.g., `ellielab.duckdns.org`.
   - Copy your token.

2. Create DuckDNS update script (`duckdns-update.sh`):
   ```bash
   #!/usr/bin/env bash
   echo url="https://www.duckdns.org/update?domains=ellielab&token=YOURTOKEN&ip=" | curl -k -o ~/duckdns/duck.log -K -
   ```

3. Make it executable:
   ```bash
   chmod +x duckdns-update.sh
   ```

4. Add to crontab (update every 5 mins):
   ```bash
   crontab -e
   */5 * * * * ~/duckdns-update.sh >/dev/null 2>&1
   ```

5. Test script:
   ```bash
   ./duckdns-update.sh
   cat ~/duckdns/duck.log
   ```

6. Confirm DuckDNS hostname resolves to your public IP:
   ```bash
   nslookup ellielab.duckdns.org
   ```

---

### 3.5. OPNsense VPN Integration with DuckDNS

1. In **VPN → Servers (WireGuard/OpenVPN)**:
   - Set hostname as `ellielab.duckdns.org`.
   - This ensures clients always reach you, even if your WAN IP changes.

2. Test from mobile device:
   - Disconnect home WiFi.
   - Connect via mobile data to `ellielab.duckdns.org`.

---

👉 At this point you’ll have:
- Pi-hole filtering DNS for your whole network.
- OPNsense forwarding DNS traffic to Pi-hole.
- DuckDNS dynamic DNS keeping your WAN address updated for remote access.
