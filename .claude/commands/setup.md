---
description: Setup — personalize this vault template for a new owner
---

# Setup — Personalize this vault

You are onboarding a **new owner** into this Obsidian vault template. Your job is to interview them, then write their context into the vault so the other skills (`/daily`, `/shutdown`, `/friday-30`, `/study`) and the Home dashboard work for *them* — not the template author.

Run this once, right after cloning. Be warm, fast, and concrete. Ask in small batches, not a wall of questions.

---

## Phase 0 — Detect the vault root

The commands in `.claude/commands/` contain a `{{VAULT_ROOT}}` placeholder. Resolve it to the **absolute path of this vault** (the current working directory / project root).

1. Determine the absolute vault root (e.g. via `pwd`).
2. Find-and-replace `{{VAULT_ROOT}}` → the real path in **every** file under `.claude/commands/`.
3. Confirm zero `{{VAULT_ROOT}}` placeholders remain.

Do this silently, then report: `✓ Vault root wired to <path>`.

---

## Phase 1 — Interview (batched)

Ask these in 3 short rounds. Accept terse answers; infer sensible defaults and confirm.

**Round 1 — Identity**
- Name (and what to call them)
- Role / what they do
- Location + timezone
- Primary language for notes & conversation

**Round 2 — Communication style**
- Tone (casual/direct? formal? bullets vs prose?)
- Technical level (assume expertise, or explain?)
- Anything you should *always* or *never* do (emojis, pushback, jokes, corrections)

**Round 3 — Focus & rhythm**
- What are they working on right now (1–3 things)?
- Work hours / when they're "off"
- Top 1–3 goals for the year
- Tools that are their source of truth (calendar, tasks, etc.)

---

## Phase 2 — Write the context

Create/overwrite these notes from the answers. Keep them tight. Tag AI-authored notes per the convention in `CLAUDE.md` (`ai-generated` / `ai-assisted`).

1. **`CLAUDE.md`** (vault root) — the master context file. Who they are, how to communicate, which context files to read at session start, vault structure, key tools. Use the existing `CLAUDE.md` as the structure to fill (if absent, create it).
2. **`0. Context/About Me/Profile.md`** — background, personality.
3. **`0. Context/Preferences/Communication.md`** — full communication preferences (from Round 2).
4. **`0. Context/Work/Role.md`** — role, teams, responsibilities.
5. **`0. Context/Active/Current Focus.md`** — what they're working on now (Round 3).
6. **Goals** — for each year-goal, create a `#goal` note from `7. Templates/Goal Template.md` so the Home "Metas" grid fills.

Overwrite the placeholder example notes that shipped with the template where they overlap (Current Focus, Role, etc.). Leave the demo books/articles/dailies unless they ask to clear them (offer to).

---

## Phase 3 — MCPs (optional)

Point them at `.claude/MCP.md` for the recommended MCP server list and how to install. Ask which they want; only the ones their workflows need. `/daily`, `/shutdown` lean on **google-workspace**; `/study` leans on **notebooklm-mcp**. Everything else is optional.

---

## Phase 4 — Wrap

- Summarize what you wrote (files touched).
- Tell them to open **`Home.md`** and enable Dataview's *JavaScript Queries* if they haven't.
- Suggest first commands: `/daily` to start a day, `/study` to spin up a course.
- Offer to clear the demo example notes if they want a blank slate.
