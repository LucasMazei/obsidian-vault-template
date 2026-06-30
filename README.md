---
tags:
  - ai-generated
---

# Vault Template

A clean starter clone of Mazei's vault structure. Open this folder as a vault in Obsidian and the **[[Home]]** dashboard works out of the box.

## What's included

- **Full folder skeleton** — every top-level folder + subfolders (empty, kept with `.gitkeep`)
- **Working Home dashboard** — `Home.md`, a `dataviewjs` homepage (greeting, vault heatmap, area cards, countdowns, habit streaks, goals, reading shelf)
- **Obsidian config** (`.obsidian/`) — plugins + theme + the `home` CSS snippet pre-enabled
- **Templates** (`7. Templates/`) — Daily Note, Book, Article, Goal, Presentation, AI Chat, Weekly Journal, New Note
- **Example notes** — demo dailies, books, goals, articles, an event, and a study course so every Home widget renders out of the box (clear them anytime)
- **Claude Code skills** (`.claude/`) — `/setup`, `/daily`, `/shutdown`, `/friday-30`, `/study` (see [`.claude/README.md`](.claude/README.md)) + recommended MCP servers ([`.claude/MCP.md`](.claude/MCP.md))

## Quick start

1. **Open in Obsidian** — open this folder as a vault. Trust plugins; enable Dataview's *JavaScript Queries* (`Settings → Dataview`).
2. **Open `Home.md`** — the dashboard should render with demo data.
3. **Run `/setup` in Claude Code** — it interviews you, writes your context (`CLAUDE.md`, `0. Context/...`, goals), and wires the skills to your vault path.
4. **Add MCPs** (optional) — copy `.mcp.json.example` → `.mcp.json`, fill in your credentials. See [`.claude/MCP.md`](.claude/MCP.md).

## Folder map

```
0. Context/         - Who you are (Profile, Work, Preferences, Active Focus)
1. Rough Notes/     - Inbox for quick captures
2. Source Materials/ - Reference material (Books, Articles, Courses, Study, ...)
3. Indexes/         - MOCs (Maps of Content)
4. Zettelkasten/    - Permanent notes
5. Daily Journal/   - Daily notes
6. Projects/        - Active project docs
7. Templates/       - Note templates
Images/             - Attachments (ships with home-banner.gif)
```

## To make the Home render

The dashboard needs these — all already configured here:

1. **Dataview** plugin enabled (`Settings → Community plugins`) with **JavaScript queries** turned on (`Settings → Dataview → Enable JavaScript Queries`).
2. The **`home`** CSS snippet enabled (`Settings → Appearance → CSS snippets`).
3. `Images/home-banner.gif` present (the banner) — swap it for your own.
4. Theme **AnuPpuccin** (bundled). The color variables in `home.css` assume a Catppuccin palette.

Most sections fail gracefully when empty — create your first daily note, book, or `#goal` note and the dashboard fills in.

## First steps

- Set Home as your start page: bookmark is pre-added; for auto-open install the *Homepage* community plugin and point it at `Home`.
- Create a daily note (`5. Daily Journal`, format `YYYY-MM-DD`, uses `7. Templates/Daily Notes Template`).
- Add a `#book` note with `status: reading` to light up the reading shelf.
- Add a `#goal` note (use `7. Templates/Goal Template`) to populate the goals grid.
