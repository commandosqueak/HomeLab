# Uptime Kuma — Monitoring
**Host:** apps (VMID 200)
**URL (now):** http://192.168.8.180:3001
**URL (later):** https://uptime.container.home.lab

## Compose
Path: `/opt/stacks/uptime-kuma/docker-compose.yml`
```yaml
version: "3.9"
services:
  uptime-kuma:
    image: louislam/uptime-kuma:latest
    container_name: uptime-kuma
    restart: unless-stopped
    volumes:
      - /opt/stacks/uptime-kuma/data:/app/data
    ports:
      - "3001:3001"
    mem_limit: 512m
```

## First Run
- Create admin; add monitors (router, apps VM, NAS, Portainer, etc.)

## Backup
Bind mount: `/opt/stacks/uptime-kuma/data/`

## Operations
```bash
cd /opt/stacks/uptime-kuma
docker compose pull && docker compose up -d
```
