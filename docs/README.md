# Runbooks Index

Repeatable, step-by-step instructions for each service. If you’re *experimenting*, log it in **Diary**; once it’s stable, document the recipe here.

## App / Service Runbooks
- [Portainer — Docker Management](portainer.md)
- [Heimdall — Homelab Homepage](heimdall.md)
- [Linkding — Bookmark Service](linkding.md)
- [Uptime Kuma — Monitoring](uptime-kuma.md)
- [Homebox — Home Inventory](homebox.md)
- [LanguageTool — Grammar Server](languagetool.md)
- [Vaultwarden (Bitwarden) — Planned](vaultwarden.md)

## Tooling / Docs
- [Obsidian Plugins Runbook](obsidian-plugins.md)

## Conventions
- Project dirs live under `/opt/stacks/<app>/` on the **apps (200)** VM.
- Compose lifecycle:
  ```bash
  cd /opt/stacks/<app>
  docker compose pull && docker compose up -d
  docker image prune -f
  ```
- Backups: bind-mount/volume paths are listed in each runbook; mirror to NAS regularly.
