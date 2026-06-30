# Shutdown

Close the workday. Review what happened, discharge every open loop, prepare for tomorrow, and cut the cord.

Combines the End of Day reflection with Cal Newport's Shutdown Ritual and Zeigarnik discharge.
The goal: after this ritual, your brain has PERMISSION to stop thinking about work.

<HARD-GATE>
Do NOT write to the journal until ALL interactive phases are complete.
Gather diffs first, interact fully, discharge loops, THEN write once.
</HARD-GATE>

## Dialogue Rules

```
TONE: Encouraging but honest — acknowledge progress, call out avoidance, help transition to personal time
- One question at a time. Evenings are low-energy.
- Use AskUserQuestion with multiple choice when possible.
- Keep messages SHORT — this is a 10-min closing ritual, not a retrospective.
- If the day went sideways, normalize it: "Rough day. What's one thing that went right?"
- KEY PRINCIPLE: Every open item must either have a plan (date + next action) or be explicitly released.
  This is what gives the brain permission to stop.
```

---

## Tool Setup

Load deferred tools at the very start:
```
ToolSearch("select:AskUserQuestion,mcp__google-workspace__get_events,mcp__google-workspace__list_tasks,mcp__google-workspace__manage_task,mcp__google-workspace__manage_event")
```

| What | How |
|------|-----|
| Calendar (today) | `mcp__google-workspace__get_events` |
| Calendar (tomorrow) | `mcp__google-workspace__get_events` |
| Tasks (list) | `mcp__google-workspace__list_tasks` |
| Tasks (update) | `mcp__google-workspace__manage_task` |
| Calendar (create) | `mcp__google-workspace__manage_event` |

**Google email:** `lucas.mazei@v4company.com`
**Task list IDs:** Pessoal = `VGZfdXE0NjZkUEplYmtTeA` · V4 Company = `N1VYWldUWmlxYmRvWXE4bg` · Delegated = `TGRUYlBncjU3UUpTYm51Sg`
**Vault root:** `{{VAULT_ROOT}}`
**Journal path:** `5. Daily Journal/YYYY-MM-DD.md`
**Mental Inventory:** `0. Context/Active/Mental Inventory.md`
**Study Queue:** `0. Context/Active/Study Queue.md`
**Telegram capture state:** `.claude/state/telegram-capture.json` (Saved Messages = capture inbox)

---

## Phase 0: Context

Before anything else, check the current time:
```
Bash: date '+%H:%M %Z' --date='TZ="America/Sao_Paulo"'
```
Use this to adapt tone (e.g., "Still early, good shutdown discipline" vs "Late night — let's wrap fast") and to calculate how long the workday actually ran.

---

## Phase 1: Gather (Silent)

Fetch everything in parallel. Don't show raw results.

1. **Morning baseline** — Read today's journal (`5. Daily Journal/YYYY-MM-DD.md`)
   - Extract morning calendar snapshot
   - Extract morning task snapshot
   - Extract "Biggest Thing Today"
   - Extract habits checklist
2. **Current tasks** — Fetch from both lists (Pessoal + V4 Company), show_completed=false
3. **Today's calendar** — Fetch today's final events
4. **Tomorrow's calendar** — Fetch tomorrow's events
5. **Tomorrow's tasks** — Tasks due tomorrow (due_max filter)
6. **Mental Inventory** — Read `0. Context/Active/Mental Inventory.md`
7. **Telegram captures** — Read `.claude/state/telegram-capture.json`. Chamar `mcp__telegram__get_messages(dialog_id=saved_messages_user_id, limit=50)` e filtrar `date > last_read_iso`. Guardar pra Phase 3c.
8. **Calculate diffs** — Compare morning baseline vs. now:
   - **Calendar:** added, cancelled, rescheduled
   - **Tasks:** completed, added, rescheduled, deleted
9. **Finance sync detection (background)** — spawn a `run_in_background` general-purpose sub-agent that runs `/finance-sync --detect` (default 5d): pulls the Telegram "Finance" group + the finance app, dedups, and returns the missing-expense list. Don't wait — poll for its result before Phase 3f.

---

## Phase 2: Reflect (One Question at a Time)

### 2a. Constraint Check (was: Biggest Thing)
- Start with the morning's `## 🎯 Constraint` (or legacy `## Biggest Thing Today`)
- "A constraint era: **{X}**. Bateu?"
- `AskUserQuestion`: **Sim, fechei** / **Parcial** / **Não bati** / **Não lembro**

**If Sim, fechei OR Parcial:**
- Extract estimate from journal — procura prefix `[Xmin]` no title da task ou linha `**Estimate raw:**` no Capacity Audit callout
- Ask actual minutes — `AskUserQuestion`: **15min** / **30min** / **45min** / **1h** / **1h30** / **2h** / **3h+**
- (For "3h+": follow-up `AskUserQuestion` aberta com valor exato)
- Compute `ratio = actual / estimate` (silent, vai pro journal pra Friday 30 ler)

**If Parcial / Não bati / Não lembro:**
- Brief follow-up: "O que aconteceu?" (texto livre opcional)

**Update `.claude/state/carry-overs.json`:**
- `Sim, fechei` → reset streak: `{task: null, streak: 0, history: [...prev, {date, result, ratio}]}`
- `Parcial` / `Não bati` → keep task, increment streak, append history (Parcial conta — Aging Rule D8)
- `Não lembro` → increment streak + flag `result_uncertain: true` no history
- Se task de hoje (constraint) ≠ task de ontem (do follow-up) → streak da ontem fica registrada como "abandonada", começa novo tracking pra task de hoje

Este state file é o que dispara a Aging Gate no /daily do dia seguinte (streak ≥3).

### 2b. Habits Check
- Show today's habits checklist (Language Learning, Reading, Exercise, Healthy Eating, Culto Familiar)
- Use `AskUserQuestion` (multiSelect: true) to mark which were completed
- Update the journal's habits section with [x] for completed ones

### 2c. Task Review
- Show the diff: what changed since morning
- Ask which tasks were completed (use `AskUserQuestion`, multiSelect: true)
- Mark completed tasks via `mcp__google-workspace__manage_task` (status: "completed")
- For uncompleted tasks due today: "These didn't get done. Reschedule to tomorrow?"
  - Reschedule via `mcp__google-workspace__manage_task` (update due date)

### 2d. Untracked Work
- "Did you get anything done that wasn't on the list?"
- If yes, capture for the journal

### 2e. Evening Check-in (Energy / Mood / Drive)
- Ask 3 scores (1-5 each) in a single `AskUserQuestion` with 3 questions:
  - **Energy** (1-5): "How's your battery now?" (1=completely drained, 5=still going strong)
  - **Mood** (1-5): "How are you feeling?" (1=rough, 5=great)
  - **Drive** (1-5): "How motivated do you feel about tomorrow?" (1=dreading it, 5=can't wait)
- Ask activity tags: "What shaped your day?" — use `AskUserQuestion` (multiSelect: true) with options:
  - Good sleep, Exercise, Deep work, Conflict/stress, Social/collaboration, Back-to-back meetings, Won a battle
- These go into the `## Evening Check-in` section in the End of Day journal block

### 2f. Quick Reflection
- "What was your biggest win today?" (one question)
- "Anything you'd do differently?" (one question)
- Keep it to these two. Don't overdo it.

---

## Phase 3: Shutdown Ritual (The Loop Discharge)

This is the critical phase. Every open item gets a plan or gets released.

### 3a. Open Loop Scan
Compile a list of ALL open items across:
- Uncompleted tasks from today (already rescheduled in 2c)
- Tasks due tomorrow
- Open loops from Mental Inventory
- Floating tasks from Mental Inventory
- On Hold items from Mental Inventory

Show the full list and say:
> "Here's everything that's open. Let's make sure each one has a plan so your brain can let go."

### 3b. Loop Discharge (for each item)
For items that DON'T already have a clear next action + date, ask:

Use `AskUserQuestion`:
- **Has a plan, I'm good** — Skip (already discharged by having a date/next action)
- **Needs a next action** — Ask: "What's the next concrete step?" → update task notes
- **Defer to later this week** — Reschedule
- **Not my problem / letting go** — Move to Mental Inventory "Cleared" section
- **Kill it** — Delete or archive

For items that already have dates and next actions, batch-confirm:
> "These already have plans: {list}. All good?"

### 3c. New loops + Telegram captures

**3c1. Telegram captures (se houver):** Se Phase 1 step 7 trouxe mensagens, surface uma por uma:
> "Você capturou no Telegram às {time}: \"{texto}\". Pra onde?"

`AskUserQuestion`: `Pessoal (GTask)` / `V4 (GTask)` / `Mental Inventory` / `Study Queue` / `Lixo`. Aplicar via `manage_task` ou append no arquivo certo.

**3c2. Brain dump final:**
> "Anything still nagging you that we haven't covered?"
- If yes → quick Worry Tree: "Can you do something about it?" → capture or discharge
- If no → proceed

**3c3. Finance Sync (auto):** Poll the background sub-agent spawned in Phase 1 step 9 (`/finance-sync --detect`, 5d). When it returns:
- If `missing_count == 0` and no flagged → one line: "💸 Finanças sincronizadas (últimos 5d) ✅" and move on.
- If there are missing/flagged expenses → show the count + total and offer to sync now:
  `AskUserQuestion`: **Sincronizar agora** (roda `/finance-sync` interativo — escolhe quais + categoria) / **Skip, sincronizo depois** / **Mudar janela**.
  - "Sincronizar agora" → invoke `/finance-sync` (interactive) inline before continuing.
  - "Skip" → note it in the journal Loop Status so it's not lost.
- This is evening / Personal window, so creating personal expenses here is fine (unlike the morning /daily).

### 3d. Tomorrow Preview
- Show tomorrow's first meeting time
- Show task count for tomorrow
- Show any early morning commitments
- "Tomorrow starts at {first event time}. You have {N} tasks due. Looks {manageable/tight}."

### 3e. The Trigger
After everything is reviewed and discharged:
> "Every open loop has a plan or has been released. Your system has it. You don't need to hold it."
>
> **Shutdown complete.**

---

## Phase 4: Output

### 4a. Update Mental Inventory
- Add new open loops / floating tasks
- Move resolved/killed items to Cleared archive
- Update "Last updated" date

### 4a1. Update Telegram capture state
Se Phase 1 step 7 trouxe mensagens, atualizar `last_read_iso` em `.claude/state/telegram-capture.json` pro ISO atual. **Não deletar** mensagens do Telegram.

### 4b. Update Journal
Append to today's journal (`5. Daily Journal/YYYY-MM-DD.md`), before `# Daily Prayer`:

```markdown
# End of Day

## 🎯 Constraint
- **Result:** {Sim, fechei / Parcial / Não bati / Não lembro} {brief note}
- **Estimate raw:** {Xmin} (do morning baseline)
- **Actual:** {Xmin} (só se Sim/Parcial)
- **Ratio:** {actual/estimate, ex 1.8x} (só se Sim/Parcial)

## Habits
- {which were completed}

## Evening Check-in
**Energy:** {N}/5 · **Mood:** {N}/5 · **Drive:** {N}/5
**Tags:** {activity tags, comma-separated}

## Day Changes

### Calendar
- Added: {events added during day}
- Cancelled: {events cancelled}
- Rescheduled: {events that moved}

### Tasks
- Completed: {tasks done}
- Added: {tasks created during day}
- Rescheduled: {tasks moved to later}

## Untracked Work
- {anything done that wasn't on the list, or "None"}

## Wins
{biggest win}

## Learnings
{what would do differently}

## Tomorrow
**First meeting:** {time} - {event}
**Tasks due:** {count}

## Loop Status
- Discharged: {N} items have plans
- Released: {N} items let go
- Killed: {N} items removed

---
*Shutdown complete — {time}*
```

### 4c. Git Commit
- `git add -A`
- `git commit -m "shutdown: YYYY-MM-DD"`
- Only if there are changes

---

## What This Skill is NOT

- **Not a performance review** — no grading, no guilt
- **Not a planning session** — tomorrow's plan happens in `/daily`
- **Not optional** — even on bad days, discharge your loops
- **Not long** — 10 minutes max, then close the laptop
- **Not just a journal entry** — the loop discharge is the whole point
