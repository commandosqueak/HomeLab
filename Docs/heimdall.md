# Heimdall — Homelab Homepage
**Host:** apps (VMID 200)
**URL (now):** http://192.168.8.180:8080
**URL (later):** https://heimdall.container.home.lab

## Compose
Path: `/opt/stacks/heimdall/docker-compose.yml`
```yaml
version: "3.9"
services:
  heimdall:
    image: linuxserver/heimdall:latest
    container_name: heimdall
    restart: unless-stopped
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
    volumes:
      - /opt/stacks/heimdall/config:/config
    ports:
      - "8080:80"
    mem_limit: 512m
```

## Notes
- Add tiles for Portainer, Linkding, Uptime Kuma, Homebox, etc.

## Backup
Bind mount: `/opt/stacks/heimdall/config/` → back up to NAS.

## Operations
```bash
cd /opt/stacks/heimdall
docker compose pull && docker compose up -d
```
