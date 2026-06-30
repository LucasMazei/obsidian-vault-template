# Daily Summary (capacity-aware)

Generate your daily summary and write it as your daily journal note. It's capacity-aware: it models your real free time against meetings before suggesting what to commit to today.

<HARD-GATE>
Required phases (não pular):
- Phase 0 (start timer + day detection)
- Phase 1 (gather)
- Phase 1.5 (calendar triage — interactive)
- Phase 1.6 (capacity calc — silent)
- Phase 1.7 (1-1 briefings spawn — background)
- Phase 2.5 (Maker AM Defense — só Tue/Thu, salvo skip-until)
- Phase 2a0 (catch-up sweep — fecha gap de fim de semana / dias pulados)
- Phase 2a/2a1 (habits + scores)
- Phase 2a2 (aging gate — força Slice/Schedule/Kill se streak ≥3)
- Phase 2b (mental inventory + on-deck audit)
- Phase 2c (telegram + brain dump)
- Phase 2d (constraint task + supporting + reality check)
- Phase 2g (calendar blocking — opt-out, default Sim)
- Phase 3a (journal write — preserve `# Notes`)
- Phase 3c (commit)
- Phase 3f (duration)

Optional (skip silenciosamente em fim de semana / survival mode):
- Phase 2e (per-task estimates) — só se faltar estimate
- Phase 3b (social media)
- Phase 3e (Friday 30 hook — só sexta)

Gather + triage everything first, interact fully, THEN write once.
</HARD-GATE>

## Dialogue Rules

```
TONE: Chief of Staff — efficient, direct, slightly pushy on priorities
- One question at a time (or batched via AskUserQuestion's 4-question screens)
- Mornings are low-energy — use multiple choice over open text
- Keep messages SHORT — 10-min ritual, not planning session
- Call out time math bluntly if it doesn't add up
- Don't lecture. Just do.
```

---

## Tool Setup

Load deferred tools at the very start:
```
ToolSearch("google-workspace calendar tasks gmail")
ToolSearch("manage_event get_drive_file_content download_chat_attachment")
```

| What | How |
|------|-----|
| Calendar (list w/ RSVP) | `mcp__google-workspace__get_events` with `detailed: true, include_attachments: true` |
| Calendar (decline) | `mcp__google-workspace__manage_event` action=`update` with attendee response or `respond_to_event` |
| Calendar (create) | `mcp__google-workspace__create_event` (visibility: private) |
| Task lists | `mcp__google-workspace__list_task_lists` |
| Tasks (list) | `mcp__google-workspace__list_tasks` (`show_assigned: true, show_hidden: true`) |
| Tasks (manage) | `mcp__google-workspace__manage_task` |
| Gmail | `mcp__google-workspace__search_gmail_messages` |
| Drive file | `mcp__google-workspace__get_drive_file_content` (for transcripts via attachment) |
| Weather | `WebFetch("https://wttr.in/{{CITY}}?format=j1")` |
| Telegram capture | `mcp__telegram__get_messages` |

**Google email:** _your email (set in `CLAUDE.md`)_
**Vault root:** `{{VAULT_ROOT}}`
**Journal path:** `5. Daily Journal/YYYY-MM-DD.md`
**Briefings path:** `5. Daily Journal/Briefings/YYYY-MM-DD/{slug}.md`
**State dir:** `.claude/state/`

**State files:**
- `telegram-capture.json` — `saved_messages_user_id, last_read_iso`
- `carry-overs.json` — `{task, first_set, streak, history[]}`
- `estimate-multiplier.json` — `{multiplier, last_calibrated, sample_size, ratio_history[]}`
- `maker-am-skip-until.json` — `{skip_until: "YYYY-MM-DD", reason}` (optional)
- `weekly-constraint.json` — written by `/friday-30`, read by `/daily` on Mondays

> ⚠️ Sempre `list_task_lists` primeiro — não hardcodar IDs.

---

## Work Hours Discipline

- **Work window:** 09:00 – 18:00
- **Personal window:** 18:00 – 20:00
- **Hard OFF:** 20:00+

Work task após 18h é **exceção, não default**. Skill atrita (`AskUserQuestion` 1-clique confirmação) mas não bloqueia.

---

## Phase 0: Context + Start Timer

1. Start timer:
   ```bash
   START_TS=$(date '+%s')
   ```
2. Hora local + dia da semana:
   ```bash
   TZ='America/Sao_Paulo' date '+%Y-%m-%d %H:%M %A'
   ```
3. **Day routing:**
   - `sábado` / `domingo` → Phase W (weekend mode, curto)
   - `Monday` → standard + read `weekly-constraint.json` se existir (sugere como constraint default)
   - `Tuesday` / `Thursday` → standard + Phase 2.5 ativa
   - `Wednesday` → standard, theme=People/Ops (só registra)
   - `Friday` → standard + Phase 3e (Friday 30 hook)
4. Adaptar saudação.

---

## Phase 1: Gather (silent, parallel)

Don't show raw results — collect.

1. **Yesterday's journal** — `5. Daily Journal/{prev_workday}.md`. Extrair: habits, constraint, sleep/energy/mood/drive, streak.
1b. **Uncaptured days (gap detection — NEW)** — listar os arquivos `5. Daily Journal/YYYY-MM-DD.md` existentes. Computar `gap_days` = todo dia-calendário entre o **último journal existente** (exclusive) e hoje (exclusive). Típico numa segunda: `[sábado, domingo]`. Numa terça normal: `[]` (só ontem, já coberto). Guarda `gap_days` pra Phase 2a0. Se um gap_day **já tem** arquivo mas com habits todos `[ ]` (ex: sábado weekend-mode em branco), inclui ele no sweep também.
2. **Last 3 dailies** — extrair scores pra trend.
3. **Weather** — `wttr.in/{{CITY}}` (set your city).
4. **Calendar events** — `get_events` com `detailed: true, include_attachments: true`, 00:00–23:59 -03:00. Pra cada event capturar: title, start, end, attendees (com responseStatus pra self), description, attachments, recurring flag.
5. **Tasks** — `list_task_lists` → cada lista com `show_completed:false, show_assigned:true, show_hidden:true`. Tag origem (`[Work]`/`[Pessoal]`/etc). **Triage "Minhas tarefas"** (deve estar vazia — se não, levanta na Phase 2b antes de seguir).
6. **Gmail** — unread count + starred count.
7. **Mental Inventory** — read `0. Context/Active/Mental Inventory.md`. Salva snapshot pra diff posterior.
8. **State files** — read todos os `.claude/state/*.json` listados acima.
9. **Telegram captures** — `get_messages(me, since=last_read_iso, limit=50)`. Guarda pra Phase 2c.

---

## Phase 1.5: Calendar Triage (interactive — NEW)

Antes de qualquer cálculo de tempo livre. Classifica cada event:

```
ATTENDING      = self_organizer OR responseStatus=accepted OR title ∈ ROUTINE_WHITELIST
SKIP           = responseStatus=declined
AMBIGUOUS      = responseStatus ∈ {needsAction, tentative} (sem flag suggest-skip)
SUGGEST-SKIP   = AMBIGUOUS + matches heuristic:
                 - recurring + sem description + sem note recente
                 - title startswith "[Daily]" / "[Weekly]" sem description
                 - "Transmissão ao vivo" / "WEBNAR" / "Webinar"
                 - title startswith "ENC:" / "FW:" / "Fwd:"
                 - ≥10 attendees AND não é organizer
                 - Work event fora window 09-18 → flag adicional 🌙
                 - Personal-style event antes 18h → flag ☀️ (manual review)
```

**Routine whitelist:** recurring personal/routine calendar blocks that should never be flagged for skipping — e.g. `Wake-up Routine`, `Study`, `Lunch`, `Workout`, `OFF`, and all-day context events. Customize this list to match your own recurring calendar entries.

**Triage interativa:**
- Se AMBIGUOUS + SUGGEST-SKIP totalizar 0 → silencia, segue.
- Senão → uma ou mais chamadas `AskUserQuestion` multiSelect com até 4 opções cada. Cada opção = um event. Label format:
  ```
  HH:MM {title} ({duration}) {flag emoji} {reason}
  ```
  Default unchecked = **não vou**.

**Após triage:**
- Pra cada event não-checado: chamar `manage_event` com `attendee.responseStatus=declined` (auto-decline conforme D1).
- Pra cada checado: marca como ATTENDING.
- Registrar lista de Declined pra subseção do journal (D29).

---

## Phase 1.6: Capacity Calc (silent — NEW)

Com Attending finalizado:

```
work_busy        = sum(duration de events ATTENDING em 09:00-18:00)
work_window      = 9h = 540min
work_free        = soma de gaps ≥50min em 09-18 entre events ATTENDING
work_capacity    = (work_free × 0.7 − 60) / 1.5

personal_busy     = sum(duration de events ATTENDING em 18:00-20:00)
personal_free     = soma de gaps ≥30min em 18-20 entre events ATTENDING
personal_capacity = (personal_free × 0.7) / 1.5
                     (sem thief tax na janela curta)
```

Se multiplier calibrado (do state file) > 1.5, usar esse no lugar de 1.5.

**Output:**
- Variável `capacity = {work: X, personal: Y, multiplier_used: M, samples: N}` pra Phase 2d/2e.
- Print silent pro user no fechamento da gather: `Capacity hoje: Work ~Xmin · Personal ~Ymin (multiplier ×M, N samples)`.

---

## Phase 1.7: 1-1 Briefings Spawn (background — NEW)

Identificar 1-1s ATTENDING:
- Title contém `1-1`, `1:1`, `[1-1]`, `[1:1]`, OU
- Exatamente 2 attendees (você + 1)
- **Excluir** se title startswith `[Daily]` / `[Weekly]` (whitelist out, D25).

Pra cada 1-1 attended:
1. Extrai counterpart (email + nome do attendee).
2. Resolve nome via memory references (CLAUDE.md, MEMORY.md → People to Know).
3. Check se counterpart é externo (email não-Work) → `AskUserQuestion`: _"Gerar briefing pra {Nome} (externo)?"_ Yes/No.
4. Check `5. Daily Journal/Briefings/{today}/{slug}.md` — se existir + < 4h velho → `AskUserQuestion`: _"Briefing existente <4h. Reusar ou regenerar?"_
5. Senão, **spawn sub-agent em background** (general-purpose, `run_in_background: true`) com prompt:
   ```
   You're generating a 1-1 briefing for the vault owner's meeting with {Nome} ({email}) at {time} today.

   Tasks:
   1. Search Google Calendar for 1-1s with {Nome} in last 30 days.
   2. For each past 1-1, pull attached transcript files (via include_attachments) and download content from Drive.
   3. grep the Obsidian vault at {{VAULT_ROOT}} for mentions of "{Nome}" in journals (5. Daily Journal/), Mental Inventory (0. Context/Active/), projects (6. Projects/) — last 60 days.
   4. Generate briefing markdown at: 5. Daily Journal/Briefings/{today}/{slug}.md
      Format (start the file with YAML frontmatter so it's tagged as AI-authored):
      ---
      tags: [ai-generated]
      ---

      ## 1-1 com {Nome} — {hoje}
      ### Último 1-1: {data} ({N dias atrás})
      ### Open follow-ups
      ### Mentions recentes no vault
      ### Sugestões de pauta

   If last 1-1 was <3 days ago: briefing curto (só follow-ups, sem dump).
   If first 1-1 ever (no history): "discovery mode" — contexto + pauta inicial.
   Latency budget: 2 min. If exceeded, write what you have and exit.
   ```
6. Não esperar aqui. Continua pra Phase 2.

**Antes do journal write (Phase 3a):** poll status dos sub-agents (até 2min total). Linkar briefings completos no journal callout. Marcar pendentes como `_pending_`.

---

## Phase 2.5: Maker AM Defense (Tue/Thu only — NEW)

Check antes:
- `day_of_week ∈ {Tuesday, Thursday}` ?
- Read `.claude/state/maker-am-skip-until.json`. Se `today < skip_until` → skip esta phase, registra no journal `_Maker AM skipped (crisis opt-out until {date}, reason: ...)_`.

**Window:** 09:00-12:00 (3h).

**Scan** (silent, com Attending finalizado):
```
meetings_in_window = ATTENDING events em 09-12 (excluindo routine whitelist)
gap_max            = maior gap contíguo ≥90min restante em 09-12
```

**Estado:**
| Estado | Critério | Skill faz |
|---|---|---|
| 🟢 Clean | `meetings_in_window == 0` | Mostra: _"Maker AM livre. Cria block tu mesmo (3h, Constraint focus)."_ |
| 🟡 Partial | `1-2 meetings, gap_max ≥90min` | Mostra: _"Maker AM tem {gap_max}min em {HH:MM-HH:MM}. Cria block tu mesmo. Quer declinar {sugestões}?"_ |
| 🔴 Broken | `gap_max < 90min` | `AskUserQuestion` multiSelect: _"Maker AM destruído por {meetings}. Quais declinar pra liberar?"_ → executa decline. Re-scan. |

**Não auto-cria block** (D22 — você cria). Só sinaliza + abre espaço.

Skip se survival mode declarado em Phase 2d (post-hoc — registra "Maker AM skipped (survival)").

---

## Phase 2: Interact

### 2a0. Catch-up Sweep (gap days — NEW, D32)

Roda **antes** da Q1 de ontem. Fecha o furo de fim de semana / dias sem run.

- Se `gap_days` (da Phase 1b) tem **só ontem** (ou está vazio) → skip esta sub-fase, segue pro 2a normal.
- Se `gap_days` tem ≥2 dias (típico segunda: sáb+dom, ou qualquer dia pulado) → **sweep obrigatório**:
  - Uma `AskUserQuestion` multiSelect **por gap day** (label `{Weekday DD/MM}`), opções = os 5 habits. Default unchecked.
  - Tom: _"Fechar o tracking de {dia}: o que rolou?"_ — rápido, sem cobrança (fim de semana é leve).
  - Pra cada gap day:
    - Se o arquivo **não existe** → cria stub mínimo (`5. Daily Journal/YYYY-MM-DD.md`) com frontmatter `tags: [ai-generated]` + só o callout `[!habits]` (resto do template é opcional em catch-up). Marca os `[x]` conforme resposta.
    - Se **existe** mas habits em branco → back-mark `- [ ] {habit}` → `- [x] {habit}` no callout. Idempotente.
  - **Domingo:** habit fica registrado (`[x]`), mas **não conta no streak** — o tracker do home já pula domingo. Marca mesmo assim pra honestidade do registro.

### 2a. Habits

`AskUserQuestion` combinado:
- Q1 (multi): "Habits ontem ({DD/MM})?" — Language Learning, Reading, Exercise, Healthy Eating
  - ⚠️ "Ontem" = `prev_workday` real (segunda → sexta). Sáb/dom já foram cobertos no 2a0 — **não** despejar fim de semana no arquivo de hoje (bug histórico: 22/06 nasceu 5/5 porque o weekend caiu no dia errado).
- Q2 (multi): "Já fez algo hoje?" — Reading, Exercise, Language Learning, Nothing yet

**Marcar os checkboxes (fix streak — D31, obrigatório):**
O home dashboard + plugin Tracker contam os `- [x]` dos dailies. A prosa/resposta não basta — sem marcar os boxes, o streak nasce zerado.
- **Ontem:** Read `5. Daily Journal/{prev_workday}.md`; pra cada habit marcado na Q1, troca `- [ ] {habit}` → `- [x] {habit}` dentro do callout `[!habits]`. (Idempotente — se já `[x]`, deixa.)
- **Hoje:** os habits da Q2 entram já como `[x]` no template da Phase 3a (resto `[ ]`).
- **Domingo NÃO conta no streak** (dia de descanso): não força marcação no domingo e a ausência de domingo não quebra a sequência. A lógica de streak do home já pula domingos — aqui é só não cobrar.

### 2a1. Sleep + Eating + Scores (4 perguntas screen 1)

Single `AskUserQuestion` com 4 questions:
1. Eating ontem: Clean / Mostly good / Not great / Terrible
2. Sleep: 7-8h / 6-7h / 5-6h / <5h
3. Energy: 5 / 4 / 3 / 2
4. Mood: 5 / 4 / 3 / 2

Drive vai numa 2ª call separada (1 pergunta): 5 / 4 / 3 / 2.

### 2a2. Aging Gate (UPGRADED)

Read `.claude/state/carry-overs.json`.

**Sempre perguntar follow-up do constraint de ontem:**
> _"Ontem o constraint era **{task}**. Bateu?"_

Opções: `Sim, fechei` / `Parcial` / `Não bati` / `Não lembro`.

**Update streak:**
- `Sim, fechei` → reset streak. `current_streak = null`.
- `Parcial` ou `Não bati` → streak += 1. (D8: Parcial conta.)
- `Não lembro` → streak += 1 + flag warn.

**Trigger Gate** (D9: streak entrando hoje == 3):
> 🚨 _"{task} é a 3ª carry-over consecutiva. Aging rule: sem 4ª. Escolhe agora:"_

`AskUserQuestion` 3 opções, sem 4ª:
- **🔪 Slice** — quebra em ≤90min, vira constraint hoje
- **📌 Schedule hard** — bloqueia slot real hoje + decline meeting
- **💀 Kill** — backlog ou delete

**Execução:**

| Opção | Skill faz |
|---|---|
| Slice | `AskUserQuestion` aberta: _"Qual o slice de ≤90min?"_ → `manage_task` create na lista origem com due hoje + estimate `[Nmin]` no title; original ganha note `blocked-by-slice: {new_task_id}`; reset streak |
| Schedule hard | Skill propõe melhor slot livre (window-aware) + sugere meeting pra declinar (do triage Step 1) → `AskUserQuestion` confirma → `create_event` + `manage_event` decline; reset streak |
| Kill | `AskUserQuestion`: _"Move pra Mental Inventory § Ideas/Someday OU delete completo?"_ (D11) → executa move (append no inventory + delete GTask) OU `manage_task` delete; reset streak |

Atualiza `carry-overs.json` no fim.

### 2b. Mental Inventory + Triagem "Minhas tarefas" + On-deck audit

1. **Triagem "Minhas tarefas"** (default list) — se Phase 1 detectou tasks lá, `AskUserQuestion` por task: Pessoal / Work / Deletar. Move/delete.
2. **Floating Tasks** do inventory → migrar pra GTasks via prompt.
3. **Open Loops** → perguntar resolvidos.
4. **On-deck audit** (NEW, D7): conta Floating Tasks + GTasks sem due (Pessoal/Work) + Open Loops ativos. Se total > 5:
   > ⚠️ _"On-deck: {N}. WIP 3/1/5 pede ≤5. Roda /mind-sweep ou mata items na Friday 30."_
   Não bloqueia, só sinaliza.
5. **Bump `Last updated`** só se algo mudou.

Skip silencioso se tudo limpo.

### 2c. Brain Dump

**2c1. Telegram captures** — pra cada msg nova: `AskUserQuestion` destino (Pessoal GTask / Work GTask / Mental Inventory / Study Queue / Lixo). Aplica.

**2c2. Brain dump** — _"Algo na cabeça que não tá em tasks/inventory/telegram?"_ Nada novo / Tem coisa.

**2c3. Update** `telegram-capture.json` com `last_read_iso = now()`.

### 2d. Constraint Task + Supporting + Reality Check (UPGRADED — WIP 3/1/5)

**2d.1 Constraint Task**

`AskUserQuestion`: _"Qual a 1 task que move a constraint hoje?"_

Opções:
- Tasks due hoje (Work + Pessoal)
- Sliced sub-task da Phase 2a2 (se houve Slice)
- "Outro" (texto livre)
- "Survival mode" (sem constraint, só passar o dia)

Se **Survival mode** → skip 2d.2/2d.3, journal marca `## 🎯 Constraint: Survival mode`. Resto do /daily segue normal sem WIP cap aplicado.

Se **Monday + weekly-constraint.json** existe → mostrar como sugestão default no topo das opções: _"(do Friday 30: {task})"_.

**2d.2 Supporting Tasks** (cap 2)

`AskUserQuestion` multiSelect, opções = tasks due restantes (sem a constraint) + "Nenhuma — só constraint".

Hard cap em 2. Se 3+ marcados:
> _"WIP cap = 3. Escolhe 2."_

**2d.3 Reality Check vs Capacity** (auto, silencia se OK)

```
multiplier  = state.estimate-multiplier.multiplier (default 1.5)
estimates_work  = sum(estimate[Work tasks selecionados])
estimates_pess = sum(estimate[Personal tasks selecionados])

budget_work   = estimates_work × multiplier
budget_pess = estimates_pess × multiplier

overshoot_work   = budget_work − capacity.work
overshoot_pess = budget_pess − capacity.personal

tolerance_work   = capacity.work × 0.15
tolerance_pess = capacity.personal × 0.15
```

| Caso | Skill faz |
|---|---|
| Ambos `overshoot ≤ tolerance` | Silencia. Segue. |
| Overshoot Work > tol | Mostra math transparente + `AskUserQuestion`: cada supporting Work + "Slice a constraint". Constraint não cai — só re-escopa. |
| Overshoot Personal > tol | Idem (só supporting Personal). |
| **Undershoot** `budget < capacity × 0.5` (D15) | Sugere: `AskUserQuestion` _"Capacity sobrando {X}min. Encaixar do Mental Inventory? Opções: {Top 3 floating/no-due tasks}"_ |

**Math display sempre transparente (D12):**
> _"Constraint 30min × 1.5 = 45 · Supp 60min × 1.5 = 90 · Total 135 · Work cap 80 → overshoot +55"_

### 2e. Per-task Estimates (só se faltar)

Pra cada task (constraint + supporting):
- Se title já tem `[Xmin]` / `[Xh]` (regex) → reusa.
- Senão `AskUserQuestion`: 15min / 30min / 1h / 2h.
- Updates title com prefix `[Xmin]` via `manage_task`.

### 2g. Calendar Blocking (UPGRADED — opt-out + window-aware)

`AskUserQuestion`:
- `Sim, bloqueia (default)` — Recommended
- `Skip — eu encaixo livre`

Se Sim → pra cada task selecionada:

**Slot picking (window-aware):**
- Work task → busca slot 09-18, gap ≥50min + 15min buffer
- Personal task → busca slot 18-20, gap ≥30min + 15min buffer
- Strategy: **Constraint = largest-fit** (D17), **Supporting = earliest-fit**
- Se não cabe na janela:
  > _"{task} não cabe em {window}. Roll pra amanhã ou aceita exceção fora-horário?"_
  Se fora-horário Work: `AskUserQuestion` 1-clique _"Confirma Work após 18h?"_ (D19).

**Create event:**
- `summary`: `🔒 [Constraint] {task title}` ou `🔒 [Supp] {task title}`
- `visibility`: `private`
- `description`: link pra GTask ID + estimate raw + budget (× mult)

Registra `Blocked?` flag por task pra coluna do journal (D5).

---

## Phase 3: Output

### 3a. Write Journal (idempotent)

Path: `5. Daily Journal/{YYYY-MM-DD}.md`

**Antes de escrever:**
1. Se file existe → Read inteiro.
2. Extrair `# Notes` em diante — preservar.
3. Reescrever com novo summary + `# Notes` injetado no final.

**Template:**

```markdown
---
tags: [ai-generated]
---

> [!habits]+ Habits
> - [{x se feito hoje, senão espaço}] Language Learning
> - [{x se feito hoje, senão espaço}] Reading
> - [{x se feito hoje, senão espaço}] Exercise
> - [{x se feito hoje, senão espaço}] Healthy Eating

> _Marcar `[x]` os habits da Q2 ("já fez hoje"). Ao preservar `# Notes` num re-run, NÃO desmarcar boxes já `[x]` por você durante o dia._

---

> [!theme]+ Day Theme
> **{Day} — {Theme}** · {sub-mode}
> _{Mon: Mixed/Reactive · Tue: Maker AM + Strategy/Product · Wed: People/Ops · Thu: Maker AM + Strategy/Product · Fri: Review + Planning}_

---

> [!checkin]+ Morning Check-in
> **Sleep:** {Xh}
> **Energy:** {N}/5 · **Mood:** {N}/5 · **Drive:** {N}/5
> **Capacity:** Work ~{X}min ({free_work} × 0.7 − 60 ÷ {mult}) · Personal ~{Y}min ({free_pess} × 0.7 ÷ {mult})
> _Multiplier: ×{M} ({N} samples). {trend flag if any}_

---

> [!nutrition]+ Healthy Eating
> **Yesterday:** {rating} {note}

---

> [!followup]+ Yesterday Constraint{ (streak: N) — only if N≥2}
> **Era:** {task}
> **Status:** {Sim/Parcial/Não/Não lembro} {action taken if aging gate fired}

---

> [!summary]- Yesterday ({MMM DD})
> **Completed:** ...
> **Missed:** ...

---

## 🎯 Constraint

> **{constraint task}**
> {rationale or "Survival mode — só passar o dia"}

{if supporting:}
**Supporting (WIP {N}/3):**
- {task1}
- {task2}

---

> [!briefings]+ 1-1 Briefings (auto-generated)
> - [[Briefings/{date}/{slug1}|{Nome1} — {HH:MM}]] · last 1-1: {N}d ago · {teaser}
> - [[Briefings/{date}/{slug2}|{Nome2} — {HH:MM}]] · _pending_ (sub-agent ainda rodando)

---

## Weather - {City}

**{Temp}** (High {Hi} / Low {Lo})
{Conditions}

---

## Calendar

### Work Meetings
| Time | Event | Notes |
| ... | ... | ... |

### Routine Blocks
- ...

### Free Blocks (post-triage)
- {HH:MM-HH:MM} ({duration})

### 🚫 Declined (after triage)
> [!declined]- {N} declined
> - {HH:MM} {title} — {reason}
> - ...

---

## Tasks

### Due Today
| Task | Estimate | List | Flag | Blocked? |
|------|----------|------|------|----------|
| ... | 30min × {M} = {budget} | Work | 🎯 constraint | ✅ 17:30 |
| ... | 30min × {M} = {budget} | Work | 🪨 supporting | ❌ não cabe 18-20 |

### Upcoming (Next 7 Days)
| Task | Due | List |
| ... | ... | ... |

---

## Language Phrases

**Deutsch:**
- "{phrase}" — {EN}

**Español:**
- "{phrase}" — {EN}

**English:**
- "{phrase}"

---

> [!info]- Morning Baseline (for EOD comparison)
> **Calendar Snapshot:**
> - HH:MM — Event
>
> **Task Snapshot:**
> - [ ] [{est}] {task} (Due: ..., List: ...)
>
> **Gmail:** {N} unread, {M} starred
>
> **Open Loops Críticos:**
> - ...

> [!capacity-audit]- Capacity Audit (for Friday 30)
> Capacity: Work {X} · Personal {Y}
> Estimates raw: Σ {raw} · Budget (× {M}): Σ {budget}
> Overshoot Work: {±Z} · Overshoot Personal: {±Z}
> Multiplier in use: {M} (samples: {N})

_⏱ Skill duration: {Xm Ys}_

---

# Notes
```

> Tudo abaixo de `# Notes` é preservado em re-runs.

### 3b. Social Media (optional)

`AskUserQuestion`: Yes / Skip (default Skip). Se Yes → `/social-media`.

### 3c. Git Commit

```bash
cd "{vault_root}" && git status --short
# se mudanças:
git add -A && git commit -m "daily: YYYY-MM-DD {day} morning snapshot"
```

### 3d. Friday 30 Hook (sexta only — NEW)

Se `day_of_week == Friday`:

`AskUserQuestion`:
- **Rodar /friday-30 agora** (recommended)
- **Agendar 16:30 hoje** → `create_event` `🔍 Friday 30 — Weekly Review`
- **Skip esta semana** → `AskUserQuestion` aberta _"Por quê?"_ → registra `_Friday 30 skipped: {reason}_` no journal pra audit

Se "Rodar agora" → após /daily fechar, invocar `/friday-30`.

### 3f. Skill Duration

```bash
END_TS=$(date '+%s')
DUR_MIN=$(( (END_TS - START_TS) / 60 ))
DUR_SEC=$(( (END_TS - START_TS) % 60 ))
```

Anotar no journal:
> _⏱ Skill duration: {DUR_MIN}m {DUR_SEC}s_

E mostrar pro user no fechamento.

---

## Phase W: Weekend Mode (sáb/dom)

Skip todo bloco operacional:
- ✅ Phase 0 + timer
- ✅ Weather
- ✅ Phase 2a / 2a1 (habits + scores)
- ✅ Phase 2a2 (yesterday follow-up se sex/sáb teve constraint)
- ✅ Phase 2c (brain dump)
- ❌ Sem triage (calendar fim de semana é dele)
- ❌ Sem capacity calc
- ❌ Sem briefings
- ❌ Sem Maker AM
- ❌ Sem constraint forçado
- ❌ Sem reality check
- ❌ Sem calendar blocking
- ❌ Sem Friday 30
- ✅ Journal write + commit + duration

Tom mais leve.

---

## What This Skill Is NOT

- **Not a weekly planner** — today only.
- **Not a reporting tool** — the journal is for you.
- **Not a slow ritual** — target 10min, mais com triage interativo. Mede e otimiza.
- **Not autonomous** — sempre confirma antes de declinar, criar event, matar task.
