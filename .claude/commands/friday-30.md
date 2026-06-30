# Friday 30 — Weekly Review

The non-negotiable 30-min Friday ritual that calibrates the whole capacity-aware system. Spawned by `/daily` on Fridays (Phase 3e). Without this, the system drifts and the multiplier stays uncalibrated.

Reference: Rule 6 of the capacity-aware Day Model — calibrate future estimates from this week's actuals.

<HARD-GATE>
Required phases (não pular):
- Phase A — Mental Inventory sweep (kill ≥14d, sem Keep ≥30d)
- Phase B — Multiplier calibration (read week's shutdown journals for ratios)
- Phase C — Lock Maker AM (Tue + Thu próxima semana)
- Phase G — Goals check-in (metas 2026: progresso + atualizar `current`)
- Phase D — Pick next week's constraint → weekly-constraint.json (pode mirar uma meta)
- Phase E — Wrap (commit + reminder próximo Friday 30)

Target duration: 30min. Mede.
</HARD-GATE>

## Dialogue Rules

```
TONE: Reflective but direct — sexta tarde, batalha do día encerrada
- Curto, decisão rápida, não procrastinação disfarçada
- AskUserQuestion sempre que cabe (low-energy session)
- Sem 4ª opção em decisões de cleanup — Aging Rule aplica aqui também
```

---

## Tool Setup

Load deferred tools:
```
ToolSearch("select:AskUserQuestion,mcp__google-workspace__create_event,mcp__google-workspace__manage_task,mcp__google-workspace__list_tasks,mcp__google-workspace__list_task_lists")
```

| What | How |
|------|-----|
| Calendar (create lock blocks) | `mcp__google-workspace__create_event` |
| Tasks (kill) | `mcp__google-workspace__manage_task` action=delete |
| Tasks (list) | `mcp__google-workspace__list_tasks` |

**Vault root:** `{{VAULT_ROOT}}`
**Mental Inventory:** `0. Context/Active/Mental Inventory.md`
**Journal week range:** segunda passada → sexta hoje
**State files:**
- `.claude/state/estimate-multiplier.json` — escrito por Phase B
- `.claude/state/weekly-constraint.json` — escrito por Phase D
- `.claude/state/carry-overs.json` — só lido (input pra Phase B)

---

## Phase 0: Start Timer + Confirm

```bash
START_TS=$(date '+%s')
TZ='America/Sao_Paulo' date '+%Y-%m-%d %H:%M %A'
```

Confirma:
> "Friday 30 — 30min pra calibrar a semana. Bora?"

`AskUserQuestion`: **Bora** / **Adiar 1h** (cria reminder + sai) / **Skip esta semana** (registra reason no journal de hoje, sai).

---

## Phase A: Mental Inventory Sweep

### A.1 Load + classify
Read `0. Context/Active/Mental Inventory.md`. Pra cada item em § **Floating Tasks**, § **Open Loops / Concerns**, § **On Hold / Waiting For**, § **Watch list**:

- Procura timestamp (do item ou `Last updated`) pra estimar idade.
- Classifica:
  - **🟢 Fresh** (<14d sem mudança) — skip, silencia
  - **🟡 Stale** (14-30d sem mudança) — força ação, opção Keep ainda existe
  - **🔴 Dead** (≥30d sem mudança) — força ação, **sem opção Keep** (D34: estrito)

### A.2 Interactive sweep
Pra cada item 🟡 ou 🔴, `AskUserQuestion`:

| Idade | Opções |
|---|---|
| 🟡 Stale | **Schedule** (cria GTask com due) / **Delegate** (move pra § On Hold com responsável) / **Kill** (move pra § Cleared) / **Keep + bump** (atualiza Last updated) |
| 🔴 Dead | **Schedule** / **Delegate** / **Kill** — **sem Keep** |

Aplicar:
- **Schedule** → `manage_task` create na lista origem (Work/Pessoal), com due date + estimate (perguntar)
- **Delegate** → mover item pra § On Hold, adicionar nota "Waiting on {pessoa}"
- **Kill** → mover pra § 🗑️ Cleared com timestamp + reason
- **Keep + bump** (só 🟡) → atualiza `Last updated` do item

### A.3 Wrap section
Mostrar: _"Sweep: {N} schedule, {M} delegate, {K} killed, {L} bumped. Inventory tem {total} items ativos agora."_

Update `Last updated:` do arquivo todo no fim.

---

## Phase B: Multiplier Calibration

### B.1 Read week's data
Pra cada dia da semana (segunda → hoje):
- Read journal: `5. Daily Journal/{YYYY-MM-DD}.md`
- Procura section `# End of Day` → `## 🎯 Constraint`
- Extrai:
  - `Result:` (só interessa **Sim, fechei** — D35)
  - `Estimate raw:` (em min)
  - `Actual:` (em min)
- Calcula `ratio = actual / estimate`

### B.2 Compute new multiplier
- Read `.claude/state/estimate-multiplier.json`. Pega `ratio_history` (array, cap 10).
- Append ratios novos da semana.
- Trunca pros últimos 10.
- `new_multiplier = média(ratio_history)`. Arredonda 1 casa decimal.
- Se `len(ratio_history) < 5` → multiplier fica em **1.5 default** (D14: sample insuficiente).

### B.3 Write + display
Write back em `estimate-multiplier.json`:
```json
{
  "multiplier": 1.8,
  "last_calibrated": "2026-05-15",
  "sample_size": 7,
  "ratio_history": [1.5, 1.7, 2.0, 1.6, 1.8, 2.2, 1.7]
}
```

Mostra: _"Multiplier: {old} → {new} ({N} samples). Tua planejamento {subestima em X% / tá calibrado / superestima}."_

Se zero samples na semana (constraint não fechou nenhuma vez): _"Sem samples — multiplier mantido em {current}. Próxima semana tenta fechar pelo menos 2 constraints pra calibrar."_

---

## Phase C: Lock Maker AM (próxima semana)

### C.1 Compute datas
```bash
NEXT_TUE=$(date -d "next tuesday" '+%Y-%m-%d')
NEXT_THU=$(date -d "next thursday" '+%Y-%m-%d')
```

### C.2 Check skip-until
Read `.claude/state/maker-am-skip-until.json` (se existir):
- Se `skip_until > NEXT_THU` → skipa Phase C inteira, registra _"Maker AM lock skipped (opt-out até {date})"_.
- Senão segue.

### C.2.5 Check existing calendar blocks (ANTES de oferecer criar)
**If you keep a recurring Maker AM focus block on your calendar (e.g. `🌊 Flow (no meetings)` 09:00–11:00, recurring Tue + Thu), that block IS the Maker AM — no need to create anything.**

Pra cada uma de `{NEXT_TUE}` e `{NEXT_THU}`, `get_events` (07:00–13:00) e procura um bloco de foco já existente (match por: `🌊 Flow`, `Flow (no meetings)`, `Maker`, ou qualquer block AM de foco sem attendees criado por você).
- Se **ambos** os dias já têm bloco → Phase C satisfeita. Registra _"Maker AM já no calendário (🌊 Flow 09–11 Tue+Thu) — nada a criar."_ e pula pra Phase D.
- Se **só um** dia tem → oferece criar só o que falta (C.3).
- Se **nenhum** → segue C.3 normal.

### C.3 Confirm before creating (D36)
Só roda se C.2.5 achou dia(s) sem bloco. `AskUserQuestion`:
> "Cria Maker AM 09-12 em **{dia(s) faltando}**?"

Opções: **Sim, cria os 2** / **Só Tue** / **Só Thu** / **Skip esta semana**.

### C.4 Create events
Pra cada confirmado:
```
create_event(
  summary: "🛠 Maker AM — Constraint focus",
  start: "{date}T09:00:00-03:00",
  end: "{date}T12:00:00-03:00",
  visibility: "private",
  description: "Defended by /friday-30 · {YYYY-MM-DD HH:MM}"
)
```

Mostra: _"Lock criado: {events}. Defendê-los na semana — se alguém pedir slot, oferece Wed/Fri."_

---

## Phase G: Goals Check-in (metas 2026)

Revisão semanal das metas anuais — mantém o `current` vivo (que alimenta o widget Metas da Home) e conecta a semana ao ano. Tom: rápido, não vira sessão de planejamento.

### G.1 Load
- Ler notas `#goal` do ano corrente em `6. Projects/The Biggest Project (me)/Yearly Planning/{YEAR}/Goals/` (frontmatter: `type`, `current`, `target`, `direction`, e opcionais `start`, `habit`, `period`, `metric`, `since`).
- **Separar:**
  - **Manuais** = metas SEM `habit` (current é editado à mão). São as que entram na atualização.
  - **Automáticas** = metas com `habit` (ex.: `metric: adherence`) — progresso vem dos dailies, NÃO perguntar/editar. Só exibir.

### G.2 Display progresso
Tabela compacta agrupada por categoria (mesma ordem do widget: Espiritual → Relacionamento → Saúde → Financeiro → Carreira → Intelectual → Pessoal):
`Meta · {current}/{target} · {%}` — calcular % igual à Home (contagem; redução usa `start`; aderência = % semanas ≥ target).
- 🔴 **Destacar paradas:** metas manuais com 0% OU sem avanço vs a última Friday 30.
- 🟢 Marcar as quase-lá (≥80%).

### G.3 Update `current` (só manuais)
`AskUserQuestion` multiSelect: _"Quais metas avançaram essa semana?"_ — opções = metas manuais (+ "Nenhuma").
- Pra cada selecionada: `AskUserQuestion` aberta _"Novo valor de `current` pra {meta}? (atual: {X}, alvo: {target})"_.
- Editar o frontmatter `current:` da nota correspondente (Read → Edit). Não tocar nas automáticas.
- Se uma meta chegou a 100% → parabenizar curto + perguntar se marca como concluída (nota pode ganhar `status: done` ou ir pra um Archive — opcional, não força).

### G.4 Bridge pra constraint
- Se houver meta(s) estratégica(s) **parada(s)**, segura pra Phase D como candidata: _"A meta '{X}' tá parada há semanas — vale virar a constraint da semana?"_

---

## Phase D: Pick Next Week's Constraint

### D.1 Surface candidates
Pull:
- Tasks com due na próxima semana (Work + Pessoal) — `list_tasks` com `due_min/max`
- Items 🟢/🟡 do Mental Inventory § Open Loops com peso estratégico
- Carry-overs ativos do `carry-overs.json` (se streak ≥1, deve ser priorizado ou matado)
- **Metas paradas da Phase G** (mover uma meta de 2026 que não anda — alta alavanca)

### D.2 Ask
`AskUserQuestion`:
> "Qual a 1 **constraint** da semana que vem? Aquela coisa que se fechar, a semana valeu."

Opções: top 3 candidatos + "Outro / escrevo agora".

### D.3 Write state file
```json
{
  "task": "Fechar Product Vision (alinhar c/ roadmap)",
  "set_date": "2026-05-15",
  "week_iso": "2026-W20",
  "estimate_hint": "120min"
}
```

`/daily` de segunda lê isso e sugere como default no Phase 2d.1.

### D.4 Optional: pre-block constraint slot
`AskUserQuestion`: _"Quer já bloquear slot pra constraint na semana? (recomendo Tue ou Thu manhã, dentro do Maker AM)"_

Se Sim → propõe slot dentro do Maker AM de Tue ou Thu, cria event linkado.

---

## Phase E: Wrap

### E.1 Summary display
```
Friday 30 fechado:
- Inventory: {N} cleaned
- Multiplier: {old} → {new}
- Maker AM: locked {Tue + Thu | só Tue | só Thu | skipped}
- Next constraint: {task}
- Duration: {Xm Ys}
```

### E.2 Append to today's journal
Append à journal de hoje (`5. Daily Journal/YYYY-MM-DD.md`) antes do `# Notes`:

```markdown
---

# Friday 30 — {YYYY-MM-DD HH:MM}

## Inventory Sweep
- Scheduled: {N}
- Delegated: {N}
- Killed: {N}
- Bumped: {N}

## Multiplier Calibration
- Previous: {old}
- New: {new} ({N} samples)
- Ratio history: {[...]}

## Maker AM Lock (próx semana)
- Tue {date}: {created/skipped}
- Thu {date}: {created/skipped}

## Metas 2026 (check-in)
- Atualizadas: {meta: X→Y, ...} (ou "nenhuma avançou")
- Paradas: {metas 🔴 ou "—"}

## Next Week's Constraint
- **{task}** ({estimate_hint})

_⏱ Friday 30 duration: {Xm Ys}_
```

### E.3 Commit
```bash
cd "{vault_root}" && git add -A && git commit -m "friday-30: YYYY-MM-DD weekly review"
```

### E.4 Reminder
> "Próximo Friday 30: **{next_friday}**. Boa semana. Schlaf gut."

---

## What This Skill Is NOT

- **Not a retro** — `/shutdown` já fez reflexão diária; aqui é calibração de sistema
- **Not optional** — sem isso, multiplier estagna, inventory engorda, constraint dilui
- **Not 1h** — 30min hard cap. Se passar, simplifica decisões (default Kill em 🔴, default Schedule em 🟡)
- **Not autonomous** — sempre confirma antes de criar event ou matar item
