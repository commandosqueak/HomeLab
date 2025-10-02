# Ellie’s HomeLab

Welcome to my living, breathing homelab docs. This repo is my **single source of truth** for everything: plans, runbooks, diagrams, and a daily diary of what I actually did.

## Repo map (high level)
```
HomeLab/
├─ README.md                # You’re here
├─ Diary/                   # Daily logs / progress (Dataview-friendly)
├─ Docs/
│  ├─ Runbook/              # One file per app/service; repeatable steps
│  ├─ Reference/            # Inventory tables, VLAN plan, diagrams
│  └─ Checklists/           # First-boot, post-install, etc.
└─ assets/                  # Images, PDFs, exports
```
> If you prefer lowercase (`docs/`, `diary/`), feel free to rename later—just update links.

## Quick start
- **Runbooks:** go to [`Docs/Runbook/`](Docs/Runbook/README.md) for app-by-app instructions.
- **Diary:** check [`Diary/`](Diary/) for daily entries and recent progress.
- **Reference:** see [`Docs/Reference/`](Docs/Reference/) for network diagrams and tables.

## Daily workflow
1. Work in Obsidian (this repo is the vault).
2. Add to **Diary** as you go (use the daily template).
3. “Promote” repeatable steps into **Runbook** pages.
4. Commit & push (Obsidian Git or GitHub Desktop).

## Obsidian plugins (core)
- Obsidian Git, Templater, Dataview, Advanced Tables, Calendar, Kanban.  
See detailed config in [`Docs/Runbook/obsidian-plugins.md`](Docs/Runbook/obsidian-plugins.md).

## Next big rocks
- OPNsense install & VLANs
- Static IPs + DNS (Pi-hole/OPNsense)
- Traefik reverse proxy + HTTPS + pretty hostnames
- Backups to NAS (+ cloud copy)
- SSH key auth everywhere (password off)

---

**Pro tip:** Keep runbooks terse and actionable. If it’s not repeatable, it belongs in the Diary.
