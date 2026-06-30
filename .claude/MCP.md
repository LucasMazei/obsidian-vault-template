# Recommended MCP servers

This vault's skills (`/daily`, `/shutdown`, `/friday-30`, `/study`) work best with a few [MCP](https://modelcontextprotocol.io) servers connected to Claude Code. None are strictly required — install only what your workflow needs.

| Server | What it does | Needed by | Priority |
|---|---|---|---|
| **google-workspace** | Google Calendar, Tasks, Gmail | `/daily`, `/shutdown` | ⭐ Core |
| **notebooklm-mcp** | NotebookLM grounding for Socratic study | `/study` | ⭐ Core |
| **telegram** | "Saved Messages" as a capture inbox | `/daily`, `/shutdown` (optional) | Optional |
| **linear-server** | Issues / projects / cycles | `/friday-30` | Optional |
| **github** | Repos, issues, PRs | — | Optional |
| **playwright** | Browser automation / web fetch | — | Optional |

> The skills degrade gracefully — if a server isn't connected, that phase is skipped. Start with the two Core servers.

---

## Install

Two ways to wire these up:

- **CLI:** `claude mcp add ...` (per the snippets below), or
- **Project file:** copy `.mcp.json.example` (vault root) → `.mcp.json`, fill in your own credentials, and reload Claude Code.

Replace every `<PLACEHOLDER>` with your own value. **Never commit real secrets** — `.mcp.json` is gitignored for that reason.

### ⭐ google-workspace
Calendar / Tasks / Gmail. Create an OAuth client in Google Cloud Console (Desktop app), enable the Calendar/Tasks/Gmail APIs.
```bash
claude mcp add google-workspace -- uvx workspace-mcp --tool-tier complete --single-user
```
Env: `GOOGLE_OAUTH_CLIENT_ID`, `GOOGLE_OAUTH_CLIENT_SECRET`, `USER_GOOGLE_EMAIL`.

### ⭐ notebooklm-mcp
Grounded study sparring. Auth once with `nlm login`.
```bash
claude mcp add notebooklm-mcp -- notebooklm-mcp
```

### telegram
Capture inbox via your own Telegram "Saved Messages". Get `API_ID` / `API_HASH` from https://my.telegram.org. Then set your user id in `.claude/state/telegram-capture.json`.
```bash
claude mcp add telegram -- uvx mcp-telegram start
```
Env: `API_ID`, `API_HASH`.

### linear-server (remote)
```bash
claude mcp add --transport http linear-server https://mcp.linear.app/mcp
```
OAuth in-browser on first use.

### github (remote)
```bash
claude mcp add --transport http github https://api.githubcopilot.com/mcp \
  --header "Authorization: Bearer <YOUR_GITHUB_TOKEN>"
```
Or use the official GitHub MCP server — see https://github.com/github/github-mcp-server.

### playwright
```bash
claude mcp add playwright -- npx @playwright/mcp@latest --caps video
```

---

After adding servers, run `/mcp` in Claude Code to check connection status.
