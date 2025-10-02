# Linkding — Bookmark Service
**Host:** apps (VMID 200)
**URL (now):** http://192.168.8.180:9090
**URL (later):** https://linkding.container.home.lab

## Compose
Path: `/opt/stacks/linkding/docker-compose.yml`
```yaml
version: "3.9"
services:
  linkding:
    image: sissbruecker/linkding:latest
    container_name: linkding
    restart: unless-stopped
    environment:
      - LD_SUPERUSER_NAME=admin
      - LD_SUPERUSER_PASSWORD=<set-strong-password>
      - TZ=Europe/London
    volumes:
      - /opt/stacks/linkding/data:/etc/linkding/data
    ports:
      - "9090:9090"
    mem_limit: 512m
```

## First Run
- Login with env superuser; change password immediately.

## Backup
Bind mount: `/opt/stacks/linkding/data/`

## Reset (fresh)
```bash
cd /opt/stacks/linkding
docker compose down
sudo rm -rf /opt/stacks/linkding/data/*
docker compose up -d
```
