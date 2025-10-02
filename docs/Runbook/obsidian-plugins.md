# Obsidian Plugins Runbook

## Core Plugins
- Obsidian Git — auto-commit/push to GitHub.
- Templater — templates for diary/runbook.
- Dataview — query notes/tasks.
- Advanced Tables — better Markdown tables.
- Calendar + Periodic Notes — daily/weekly notes.
- Kanban — backlog/doing/blocked/done.

## Nice-to-have
- QuickAdd, Linter, Excalidraw, Outliner, Buttons/Commander.

## Minimal Config

### Obsidian Git
- Auth via SSH key (recommended).
- Auto-commit on close or every 10–15 min.
- Commit msg: `docs: {{date}} update`.

### Templater
Template folder: `templates/`

Diary template (`templates/diary-daily.md`):
```markdown
# <% tp.date.now("YYYY-MM-DD") %>
## Progress
-
## Notes
-
## Next Steps
-
```

VM postinstall (`templates/vm-postinstall.md`):
```markdown
# VM Post-Install — <% tp.file.title %>
- [ ] Install openssh-server, enable SSH
- [ ] Create user + sudo
- [ ] Set password
- [ ] (Optional) qemu-guest-agent
- [ ] Add to DNS
- [ ] Snapshot baseline

### Commands
```bash
sudo apt update && sudo apt install -y openssh-server qemu-guest-agent
sudo systemctl enable --now ssh qemu-guest-agent
sudo adduser ellie && sudo usermod -aG sudo ellie
```
```

### Dataview Snippets
Open tasks:
```dataview
TASK FROM ""
WHERE !completed
SORT due ASC
```

Diary rollup:
```dataview
TABLE file.day AS Date
FROM "diary"
SORT file.day DESC
```

### Kanban
Create `diary/Kanban.md` → convert to Kanban with columns: Backlog / Doing / Blocked / Done.
