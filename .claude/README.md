# Claude Code skills for this vault

This vault ships with slash commands (skills) in `.claude/commands/`. Run them from Claude Code inside the vault.

## Start here

- **`/setup`** — run this **first**, right after cloning. Interviews you, writes your context into the vault (`CLAUDE.md`, `0. Context/...`, goals), and wires the `{{VAULT_ROOT}}` placeholder in the other commands to your real path.

## Daily / weekly rhythm

- **`/daily`** — capacity-aware morning summary: calendar, tasks, carry-overs, the one constraint to move today.
- **`/shutdown`** — end-of-day: capture loose threads, update carry-overs, close the day.
- **`/friday-30`** — weekly review: calibrate your estimate multiplier, pick next week's single constraint.
- **`/study`** — Socratic sparring inside a stateful course, grounded in NotebookLM.

## State

`.claude/state/*.json` holds rolling state the skills read/write (carry-overs, estimate multiplier, weekly constraint, study-notebook map, telegram capture cursor). They ship **empty** — the skills populate them as you go.

## MCP servers

See **[`MCP.md`](./MCP.md)** for the recommended Model Context Protocol servers and how to install them. `/daily` + `/shutdown` lean on **google-workspace**; `/study` leans on **notebooklm-mcp**.

---

> ⚠️ **Heads-up — these are real, opinionated workflows.** The command bodies were written for a specific person (an engineering manager) and still mention that context, language (PT/EN mix), and habits. Treat them as worked examples: run `/setup`, then edit the command files to fit how *you* actually work.
