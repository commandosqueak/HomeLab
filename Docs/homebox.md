# Homebox — Home Inventory
**Host:** apps (VMID 200)
**URL (now):** http://192.168.8.180:7745
**URL (later):** https://homebox.container.home.lab

## Compose
Path: `/opt/stacks/homebox/docker-compose.yml`
```yaml
version: "3.9"
services:
  homebox:
    image: ghcr.io/hay-kot/homebox:latest
    container_name: homebox
    restart: unless-stopped
    environment:
      - HBOX_OPTIONS__WEB__BASEURL=http://192.168.8.180:7745
      - TZ=Europe/London
    volumes:
      - /opt/stacks/homebox/data:/data
    ports:
      - "7745:7745"
    mem_limit: 512m
```

## First Run
- Create initial user via UI.

## Backup
Bind mount: `/opt/stacks/homebox/data/`

## Operations
```bash
cd /opt/stacks/homebox
docker compose pull && docker compose up -d
```
