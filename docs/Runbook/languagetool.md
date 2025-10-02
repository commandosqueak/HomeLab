# LanguageTool — Grammar Server
**Host:** apps (VMID 200)
**URL:** http://192.168.8.180:8010 (API at `/v2`)
**Later:** https://lt.container.home.lab (via Traefik)

## Compose
Path: `/opt/stacks/languagetool/docker-compose.yml`
```yaml
version: "3.9"
services:
  languagetool:
    image: silviof/docker-languagetool:latest
    container_name: languagetool
    restart: unless-stopped
    environment:
      - Java_Xms=256m
      - Java_Xmx=2048m
      - LT_FEATURE_OFFICE=true
      - LT_FEATURE_API=true
      - LT_SLEEP=false
    ports:
      - "8010:8010"
    mem_limit: 2300m
```

## Browser Integration
- Extension “Local server” expects localhost. Workarounds until Traefik/DNS:
  - SSH tunnel: `ssh -N -L 8010:localhost:8010 ellie@192.168.8.180`
  - Or run a local LanguageTool Docker on your PC.

## Operations
```bash
cd /opt/stacks/languagetool
docker compose pull && docker compose up -d
```
