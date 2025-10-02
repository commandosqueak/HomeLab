# Portainer — Docker Management
**Host:** apps (VMID 200) — Debian 12
**URL (now):** https://192.168.8.180:9443
**URL (later):** https://portainer.container.home.lab

## Compose
Path: `/opt/stacks/portainer/docker-compose.yml`
```yaml
version: "3.9"
volumes:
  portainer_data:
services:
  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    restart: unless-stopped
    ports:
      - "9443:9443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data
    mem_limit: 512m
```

## First Run
1. Open `:9443`, create admin user.
2. **Environments →** ensure `local` (Docker, `unix:///var/run/docker.sock`) exists. Remove any Podman envs.

## Backup
Named volume: `portainer_data`
```bash
docker run --rm -v portainer_portainer_data:/data -v $(pwd):/backup busybox tar czf /backup/portainer_data.tgz /data
```

## Operations
```bash
cd /opt/stacks/portainer
docker compose pull && docker compose up -d
docker image prune -f
```
