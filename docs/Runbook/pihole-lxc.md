# Pi-hole — LXC Runbook (CT 230)

---

## 9) Unbound (recursive) with Pi-hole — Recommended

Make Pi-hole independent and privacy‑friendly by using **Unbound** as a local recursive resolver.

### Install & configure Unbound (inside the CT)
```bash
apt update
apt install -y unbound
curl -o /var/lib/unbound/root.hints https://www.internic.net/domain/named.root
```

Create `/etc/unbound/unbound.conf.d/pi-hole.conf`:
```conf
server:
  verbosity: 0
  interface: 127.0.0.1
  port: 5335
  do-ip4: yes
  do-ip6: no
  do-udp: yes
  do-tcp: yes
  root-hints: "/var/lib/unbound/root.hints"
  harden-glue: yes
  harden-dnssec-stripped: yes
  qname-minimisation: yes
  edns-buffer-size: 1232
  prefetch: yes
  aggressive-nsec: yes
  cache-min-ttl: 60
  cache-max-ttl: 86400
  access-control: 127.0.0.0/8 allow
  auto-trust-anchor-file: "/var/lib/unbound/root.key"
  private-address: 10.0.0.0/8
  private-address: 172.16.0.0/12
  private-address: 192.168.0.0/16
  private-address: 169.254.0.0/16
  private-address: 127.0.0.0/8
  private-address: ::ffff:0:0/96
```

Init DNSSEC key & start:
```bash
unbound-anchor -a /var/lib/unbound/root.key
chown unbound:unbound /var/lib/unbound/root.key
systemctl enable --now unbound
```

### Point Pi-hole to Unbound
- Pi-hole → **Settings → DNS → Upstream**: set **Custom 1 (IPv4)** to `127.0.0.1#5335` and **untick others**.
- **Disable Pi-hole DNSSEC** (Unbound already validates).
- Apply, then:
```bash
systemctl restart unbound
pihole restartdns
```

### Tests
```bash
dig @127.0.0.1 -p 5335 example.com +short   # Unbound directly
dig @127.0.0.1 -p 53 example.com +short     # Through Pi-hole
dig +dnssec @127.0.0.1 -p 5335 dnssec-failed.org  # should SERVFAIL
```

### (Optional) Keep root hints fresh (monthly)
```bash
echo '0 4 1 * * root curl -o /var/lib/unbound/root.hints https://www.internic.net/domain/named.root && systemctl restart unbound' | sudo tee /etc/cron.d/unbound-hints
```
