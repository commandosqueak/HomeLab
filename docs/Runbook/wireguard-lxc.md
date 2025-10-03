# WireGuard VPN — LXC Runbook (CT 233)

**Goal:** Lightweight, fast VPN to your homelab.
- **CTID:** 233
- **Hostname:** `wireguard`
- **Type:** LXC (Debian/Ubuntu template)
- **CPU/RAM:** 1 vCPU / 256–512 MB
- **Disk:** 2–4 GB
- **Network (now):** `vmbr0` untagged, static `192.168.8.184/24`, GW `192.168.8.1`
- **Network (later):** VLAN 10 → `10.20.10.24`
- **WAN port (later on OPNsense):** UDP **51820**

---

## 1) Create CT in Proxmox

**Recommended CT options**
- Template: Debian 12
- **Unprivileged:** Yes
- **Features:** `keyctl=1`
- **Network:** `vmbr0`, static `192.168.8.184/24`, GW `192.168.8.1`
- **DNS:** leave default or point at Pi-hole
- Start CT → Console.

> Later (after VLANs): set **VLAN Tag = 10** and change IP to `10.20.10.24/24`, GW `10.20.10.1`.

---

## 2) Base packages & sysctl (inside CT)
```bash
apt update && apt -y upgrade
apt install -y wireguard qrencode iproute2 nftables
# enable IPv4 forwarding inside the CT
grep -q '^net.ipv4.ip_forward=1' /etc/sysctl.conf ||   echo 'net.ipv4.ip_forward=1' >> /etc/sysctl.conf
sysctl -p
```

---

## 3) Server keys & config
```bash
umask 077
wg genkey | tee /etc/wireguard/server.key | wg pubkey > /etc/wireguard/server.pub
SERVER_PRIV=$(cat /etc/wireguard/server.key)
SERVER_PUB=$(cat /etc/wireguard/server.pub)
```

Create `/etc/wireguard/wg0.conf`:
```bash
cat >/etc/wireguard/wg0.conf <<'CFG'
[Interface]
Address = 10.6.0.1/24
ListenPort = 51820
PrivateKey = REPLACE_SERVER_PRIV
# NAT for VPN clients (nftables)
PostUp = nft add table inet wg; nft 'add chain inet wg postnat { type nat hook postrouting priority 100; }'; nft add rule inet wg postnat oifname "eth0" masquerade
PostDown = nft delete table inet wg
CFG
```

Patch in the private key:
```bash
sed -i "s|REPLACE_SERVER_PRIV|$SERVER_PRIV|" /etc/wireguard/wg0.conf
```

Enable WireGuard:
```bash
systemctl enable --now wg-quick@wg0
wg show
```

---

## 4) Add a peer (example: your phone)
Generate keys:
```bash
wg genkey | tee /etc/wireguard/phone.key | wg pubkey > /etc/wireguard/phone.pub
PHONE_PRIV=$(cat /etc/wireguard/phone.key)
PHONE_PUB=$(cat /etc/wireguard/phone.pub)
```

Append peer to the server:
```bash
cat >>/etc/wireguard/wg0.conf <<'CFG'

[Peer]
# Ellie Phone
PublicKey = REPLACE_PHONE_PUB
AllowedIPs = 10.6.0.2/32
PersistentKeepalive = 25
CFG

sed -i "s|REPLACE_PHONE_PUB|$PHONE_PUB|" /etc/wireguard/wg0.conf
systemctl restart wg-quick@wg0
```

Create the client config `/root/phone.conf`:
```bash
cat >/root/phone.conf <<'CFG'
[Interface]
PrivateKey = REPLACE_PHONE_PRIV
Address = 10.6.0.2/32
DNS = 10.20.10.21   # later: Pi-hole; for now use 192.168.8.181 or any resolver

[Peer]
PublicKey = REPLACE_SERVER_PUB
Endpoint = your-duckdns-subdomain.duckdns.org:51820
AllowedIPs = 0.0.0.0/0, ::/0
PersistentKeepalive = 25
CFG

sed -i "s|REPLACE_PHONE_PRIV|$PHONE_PRIV|" /root/phone.conf
sed -i "s|REPLACE_SERVER_PUB|$SERVER_PUB|" /root/phone.conf
```

Show QR to import on the phone:
```bash
qrencode -t ansiutf8 < /root/phone.conf
```

---

## 5) OPNsense (when you’re ready)
- **Port forward** UDP **51820** from WAN → `wireguard` CT IP (now `192.168.8.184`, later `10.20.10.24`).
- Add a **firewall rule** on LAN to allow `10.6.0.0/24` → desired VLANs (LAN/IoT).
- Optional: create an **alias** for `WG_NET = 10.6.0.0/24` for tidy rules.

---

## 6) Test flow
1. On phone: import QR → connect.
2. On CT: `wg show` → you should see **latest handshake** timestamps.
3. From phone: ping `192.168.8.181` (Pi-hole) and `homeassistant:8123` (when DNS is set).
4. Browse: all traffic should route via WG (since `AllowedIPs = 0.0.0.0/0`).

---

## 7) Troubleshooting
- **No handshake:** Check OPNsense port-forward; confirm DuckDNS resolves to your WAN.
- **Handshake OK, no internet:** Ensure `ip_forward=1` and `nft` masquerade rule exists (`nft list ruleset`).
- **Slow/broken sites:** Set client MTU to **1420** (add `MTU = 1420` under `[Interface]` in the client).
- **DNS weirdness:** set `DNS` in client to your Pi-hole IP (current or future) or another known resolver.

---

## 8) Later polish
- Put WG on its **own VLAN** if you want stricter policy.
- Add more peers by repeating the keygen + config steps.
- Move CT to `VLAN 10` and IP `10.20.10.24` when the new network is live.
