# Vaultwarden (Bitwarden) — Planned
**Host:** apps (VMID 200)
**URL (later):** https://vault.container.home.lab (via Traefik)

## Compose (when ready)
Path: `/opt/stacks/vaultwarden/docker-compose.yml`
```yaml
version: "3.9"
services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    restart: unless-stopped
    environment:
      - ADMIN_TOKEN=<long-random-string>
      - SIGNUPS_ALLOWED=false
      - DOMAIN=http://10.20.10.20:8081
      - TZ=Europe/London
    volumes:
      - /opt/stacks/vaultwarden/data:/data
    ports:
      - "8081:80"
    mem_limit: 512m
```

## Notes
- Move to Traefik + HTTPS once OPNsense/static IP are set.
- Add DNS `vault.container.home.lab` to Pi-hole/OPNsense.
