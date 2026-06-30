---
tags: [ai-generated]
cssclasses: [homepage]
obsidianUIMode: preview
---

```dataviewjs
// ===== Greeting (muda por hora do dia) =====
try {
  const DT = dv.luxon.DateTime;
  const now = DT.now().setLocale("pt-BR");
  const h = now.hour;
  let g = "Boa noite";
  if (h >= 5 && h < 12) g = "Bom dia";
  else if (h >= 12 && h < 18) g = "Boa tarde";
  const cap = s => s.charAt(0).toUpperCase() + s.slice(1);
  const weekday = cap(now.toFormat("cccc"));
  const dateStr = now.toFormat("d 'de' LLLL 'de' yyyy");
  const week = now.toFormat("'Semana' WW");
  // banner GIF (cyberpunk) — overlay escuro embaixo p/ legibilidade + fade no fundo da página
  let bannerStyle = "";
  try {
    const img = app.vault.getAbstractFileByPath("Images/home-banner.gif");
    if (img) {
      const url = app.vault.getResourcePath(img);
      bannerStyle = ` style="background-image: linear-gradient(180deg, rgba(0,0,0,.15) 0%, rgba(0,0,0,.1) 40%, var(--background-primary) 98%), url('${url}'); background-size: cover; background-position: center 35%;"`;
    }
  } catch (_) {}
  const c = dv.container.createDiv();
  c.innerHTML = `
    <div class="hp-banner"${bannerStyle}></div>
    <div class="hp-greeting">${g}, Mazei <span class="spin">◎</span></div>
    <div class="hp-sub">${weekday}, ${dateStr} · ${week}</div>
    <div class="hp-quote">"What's the one task that moves the constraint today?"</div>`;
} catch(e) { dv.paragraph("⚠️ greeting: " + e.message); }
```

```dataviewjs
// ===== Atividade do vault (stats + heatmap de notas criadas/dia) =====
try {
  const DT = dv.luxon.DateTime;
  const all = dv.pages().array();
  const counts = {};
  for (const p of all) { const d = p.file.cday; if (!d) continue; const k = d.toISODate(); counts[k] = (counts[k] || 0) + 1; }
  const today = DT.now().startOf("day");
  let d = today.minus({ days: 363 });
  d = d.minus({ days: d.weekday % 7 }); // alinha início num domingo (GitHub-style)
  let total = 0; const cells = [];
  while (d <= today) {
    const k = d.toISODate(); const n = counts[k] || 0; total += n;
    const lvl = n === 0 ? 0 : n <= 1 ? 1 : n <= 3 ? 2 : n <= 6 ? 3 : 4;
    cells.push(`<i class="l${lvl}" title="${k}: ${n} nota${n === 1 ? "" : "s"}"></i>`);
    d = d.plus({ days: 1 });
  }
  const weekAgo = today.minus({ days: 7 });
  const createdWeek = all.filter(p => p.file.cday && p.file.cday >= weekAgo).length;
  const booksRead = dv.pages("#book").where(b => b.status === "read").length;
  const lessons = all.filter(p => (p.file.folder || "").includes("/learning-records")).length;
  const journalDays = dv.pages('"5. Daily Journal"').where(p => p.file.day).length;
  const stat = (n, l) => `<div class="hp-stat"><div class="n">${n}</div><div class="l">${l}</div></div>`;
  const c = dv.container.createDiv();
  c.innerHTML = `<div class="hp-label">Atividade do vault</div>`
    + `<div class="hp-stats">${stat(all.length, "notas")}${stat(createdWeek, "criadas 7d")}${stat(journalDays, "dailies")}${stat(booksRead, "livros lidos")}${stat(lessons, "lessons")}</div>`
    + `<div class="hp-heat-wrap"><div class="hp-heat">${cells.join("")}</div>`
    + `<div class="hp-heat-legend"><span>${total} notas no último ano</span><span class="scale">menos <i class="l0"></i><i class="l1"></i><i class="l2"></i><i class="l3"></i><i class="l4"></i> mais</span></div></div>`;
} catch(e) { dv.paragraph("⚠️ stats: " + e.message); }
```

```dataviewjs
// ===== Áreas (cards de navegação) =====
try {
  const C = { peach:"#fab387", mauve:"#cba6f7", blue:"#89b4fa", green:"#a6e3a1", yellow:"#f9e2af", red:"#f38ba8", teal:"#94e2d5" };
  const areas = [
    ["🎯", "Foco agora",       "Current Focus",   C.peach],
    ["🧠", "Mental Inventory",  "Mental Inventory", C.mauve],
    ["💼", "Trabalho (V4)",     "Role",            C.blue],
    ["📥", "Study Queue",       "Study Queue",     C.teal],
    ["🏆", "Metas 2026",        "Goals Dashboard", C.yellow],
    ["🫀", "Saúde",             "Health",          C.red],
  ];
  const all = dv.pages();
  const card = ([ico, ttl, target, ac]) => {
    const p = all.where(x => x.file.name === target).first();
    const meta = p ? "atualizado " + p.file.mtime.toFormat("dd/MM") : "";
    return `<a class="hp-card internal-link" href="${target}" data-href="${target}" style="--ac:${ac}">
      <span class="ico">${ico}</span><span class="ttl">${ttl}</span><span class="meta">${meta}</span></a>`;
  };
  const c = dv.container.createDiv();
  c.innerHTML = `<div class="hp-label">Áreas</div><div class="hp-grid">${areas.map(card).join("")}</div>`;
} catch(e) { dv.paragraph("⚠️ áreas: " + e.message); }
```

```dataviewjs
// ===== Contagem regressiva (qualquer nota com event_date) =====
try {
  const DT = dv.luxon.DateTime;
  const today = DT.now().startOf("day");
  const toDT = v => (v && v.toMillis) ? v : DT.fromISO(String(v));
  const raw = dv.pages().where(p => p.event_date).array()
    .map(p => { const d = toDT(p.event_date).startOf("day"); return { p, d, days: Math.round(d.diff(today, "days").days) }; })
    .filter(e => e.days >= 0);
  // dedupe por evento+data (mantém o card mais completo — com category)
  const seen = new Map();
  for (const e of raw) {
    const key = `${e.p.event ?? e.p.file.name}|${e.d.toISODate()}`;
    const cur = seen.get(key);
    if (!cur || (e.p.category && !cur.p.category)) seen.set(key, e);
  }
  const evs = [...seen.values()].sort((a, b) => a.days - b.days).slice(0, 4);
  const card = e => {
    const big = e.days === 0 ? "hoje" : e.days;
    const unit = e.days === 0 ? "" : "<small> dias</small>";
    return `<a class="hp-cd-c internal-link" href="${e.p.file.name}" data-href="${e.p.file.path}">
      <div class="d">${big}${unit}</div>
      <div class="e">${e.p.event ?? e.p.file.name}</div>
      <div class="dt">${e.d.toFormat("dd/MM/yy")}${e.p.category ? " · " + e.p.category : ""}</div></a>`;
  };
  const c = dv.container.createDiv();
  if (evs.length) c.innerHTML = `<div class="hp-label">Contagem regressiva</div><div class="hp-cd">${evs.map(card).join("")}</div>`;
} catch(e) { dv.paragraph("⚠️ countdown: " + e.message); }
```

```dataviewjs
// ===== Hábitos (streak ao vivo, dos daily journals) =====
try {
  const DT = dv.luxon.DateTime;
  const habits = [["🏋️","Exercise"],["🗣️","Language Learning"],["📖","Reading"],["🥗","Healthy Eating"],["🙏","Culto Familiar"]];
  const days = dv.pages('"5. Daily Journal"').where(p => p.file.day).array()
    .map(p => ({ date: p.file.day.toISODate(), done: (p.file.tasks || []).filter(t => t.completed).map(t => t.text.trim()) }));
  const doneSet = name => new Set(days.filter(d => d.done.some(t => t === name || t.startsWith(name))).map(d => d.date));
  const streak = set => {
    let s = 0, d = DT.now().startOf("day");
    // não penaliza hoje ainda-não-feito
    if (!set.has(d.toISODate())) d = d.minus({ days: 1 });
    while (true) {
      if (d.weekday === 7) { d = d.minus({ days: 1 }); continue; } // domingo = descanso, pula
      if (set.has(d.toISODate())) { s++; d = d.minus({ days: 1 }); }
      else break;
    }
    return s;
  };
  // maior streak histórico (domingo é ponte, não quebra)
  const maxStreak = set => {
    const dates = [...set].sort();
    if (!dates.length) return 0;
    let d = DT.fromISO(dates[0]).startOf("day"), end = DT.now().startOf("day"), cur = 0, max = 0;
    while (d <= end) {
      if (d.weekday === 7) { d = d.plus({ days: 1 }); continue; }
      if (set.has(d.toISODate())) { cur++; if (cur > max) max = cur; } else cur = 0;
      d = d.plus({ days: 1 });
    }
    return max;
  };
  const LADDER = [15, 30, 60, 120, 240, 365];
  const today = DT.now().toISODate();
  const isSunday = DT.now().weekday === 7;
  const last30 = set => {
    let n = 0, dd = DT.now().startOf("day");
    for (let i = 0; i < 30; i++) { if (set.has(dd.toISODate())) n++; dd = dd.minus({ days: 1 }); }
    return n;
  };
  const chip = ([ico, name]) => {
    const set = doneSet(name);
    const s = streak(set);
    const max = maxStreak(set);
    const l30 = last30(set);
    const on = set.has(today);
    const status = on ? "✓ feito hoje" : (isSunday ? "🌙 domingo · descanso" : "pendente hoje");
    const next = LADDER.find(m => m > s);
    const goal = next ?? 365;
    const pct = Math.min(100, Math.round((s / goal) * 100));
    const goalTxt = next ? `próx. <b>${goal}d</b>` : `🔥 <b>365d+</b> lendário`;
    return `<div class="hp-habit">
      <div class="h-top"><span class="h-name">${ico} ${name}</span><span class="h-dot ${on?"on":""}" title="${status}"></span></div>
      <div class="h-streak">${s}<small> dias</small></div>
      <div class="h-bar"><span style="width:${pct}%"></span></div>
      <div class="h-meta">${goalTxt} · 🏆 ${max}d · 📅 ${l30}/30</div>
      <div class="h-sub">${status}</div></div>`;
  };
  const c = dv.container.createDiv();
  c.innerHTML = `<div class="hp-label">Hábitos · streak</div><div class="hp-habits">${habits.map(chip).join("")}</div>`;
} catch(e) { dv.paragraph("⚠️ hábitos: " + e.message); }
```

```dataviewjs
// ===== Metas pessoais (notas #goal do ano) =====
try {
  const YEAR = dv.luxon.DateTime.now().year;
  const TYPE_COLOR = {
    "Espiritual": "#cba6f7", "Saúde": "#a6e3a1", "Carreira": "#89b4fa",
    "Financeiro": "#f9e2af", "Relacionamento": "#f38ba8", "Pessoal": "#94e2d5",
    "Intelectual": "#fab387",
  };
  const TYPE_ORDER = ["Espiritual", "Relacionamento", "Saúde", "Financeiro", "Carreira", "Intelectual", "Pessoal"];
  const rank = t => { const i = TYPE_ORDER.indexOf(t); return i === -1 ? 99 : i; };
  const goals = dv.pages("#goal")
    .where(g => !g.file.path.startsWith("7. Templates") && (!g.year || g.year === YEAR))
    .array()
    .sort((a, b) => rank(a.type) - rank(b.type) || dv.compare(a.file.name, b.file.name));
  // metas baseadas em hábito (dailies) — contagem viva
  const DT = dv.luxon.DateTime;
  let habitDays = null;
  const habitSet = name => {
    if (!habitDays) habitDays = dv.pages('"5. Daily Journal"').where(p => p.file.day).array()
      .map(p => ({ iso: p.file.day.toISODate(), done: (p.file.tasks || []).filter(t => t.completed).map(t => t.text.trim()) }));
    return new Set(habitDays.filter(d => d.done.some(t => t === name || t.startsWith(name))).map(d => d.iso));
  };
  const habitCount = (name, days) => {
    const set = habitSet(name);
    let n = 0, dd = DT.now().startOf("day");
    for (let i = 0; i < days; i++) { if (set.has(dd.toISODate())) n++; dd = dd.minus({ days: 1 }); }
    return n;
  };
  // aderência: % de semanas COMPLETAS desde `since` com >= target ocorrências
  const adherence = g => {
    const set = habitSet(g.habit);
    const tgt = Number(g.target) || 1;
    const since = g.since ? (g.since.startOf ? g.since : DT.fromISO(String(g.since))) : DT.fromISO(`${YEAR}-01-01`);
    const today = DT.now().startOf("day");
    let wk = since.startOf("week"), hits = 0, denom = 0;
    while (wk.plus({ days: 7 }) <= today) { // só semanas já fechadas
      let c = 0;
      for (let i = 0; i < 7; i++) if (set.has(wk.plus({ days: i }).toISODate())) c++;
      denom++; if (c >= tgt) hits++;
      wk = wk.plus({ days: 7 });
    }
    return { hits, denom, pct: denom ? Math.round((hits / denom) * 100) : 0 };
  };
  const isAdh = g => g.habit && String(g.metric).toLowerCase() === "adherence";
  const curOf = g => g.habit
    ? habitCount(g.habit, String(g.period).toLowerCase() === "week" ? 7 : 30)
    : (Number(g.current) || 0);
  const pctOf = g => {
    if (isAdh(g)) return adherence(g).pct;
    const cur = curOf(g), tgt = Number(g.target) || 0;
    const st = (g.start !== undefined && g.start !== null) ? Number(g.start) : null;
    if (String(g.direction).toLowerCase() === "minimize") {
      if (st !== null && st !== tgt) return Math.round(((st - cur) / (st - tgt)) * 100); // baseline-aware (ex.: 30%→25%)
      return cur <= tgt ? 100 : Math.round((tgt / cur) * 100);
    }
    return tgt ? Math.round((cur / tgt) * 100) : 0;
  };
  const metaOf = g => {
    if (isAdh(g)) { const a = adherence(g); return a.denom ? `${a.hits} / ${a.denom} semanas ≥${g.target}` : `começando · semana 1`; }
    if (g.habit) return `${curOf(g)} / ${g.target}${String(g.period).toLowerCase() === "week" ? "/sem" : "/mês"}`;
    return `${curOf(g)} / ${g.target ?? "?"}`;
  };
  const card = g => {
    const pct = Math.max(0, Math.min(100, pctOf(g)));
    const ac = TYPE_COLOR[g.type] || "var(--interactive-accent)";
    const done = pct >= 100;
    return `<div class="hp-goal" style="--ac:${ac}">
      <div class="g-top"><span class="g-type">${g.type ?? "Meta"}</span><span class="g-pct">${done ? "✅ 100%" : pct + "%"}</span></div>
      <a class="g-name internal-link" href="${g.file.name}" data-href="${g.file.path}">${g.file.name}</a>
      <div class="h-bar"><span style="width:${pct}%"></span></div>
      <div class="g-meta">${metaOf(g)}</div></div>`;
  };
  const c = dv.container.createDiv();
  c.innerHTML = goals.length
    ? `<div class="hp-label">Metas ${YEAR} · <span class="count">${goals.length}</span></div><div class="hp-goals">${goals.map(card).join("")}</div>`
    : `<div class="hp-label">Metas ${YEAR}</div><div class="hp-quote" style="text-align:left">Nenhuma meta #goal pra ${YEAR} ainda — cria via o template em 7. Templates/Goal Template.</div>`;
} catch(e) { dv.paragraph("⚠️ metas: " + e.message); }
```

```dataviewjs
// ===== Estudando agora (cursos study-v3: 1 card por MISSION) =====
try {
  const BASE = "2. Source Materials/Study";
  const pages = dv.pages(`"${BASE}"`).array();
  // agrupa por curso (1ª pasta abaixo de Study/)
  const byCourse = {};
  for (const p of pages) {
    const rel = (p.file.folder || "").replace(BASE + "/", "").replace(BASE, "");
    const course = rel.split("/")[0];
    if (!course) continue;
    (byCourse[course] ??= []).push(p);
  }
  const cap = s => s.charAt(0).toUpperCase() + s.slice(1);
  const pretty = s => s.split(/[-_]/).map(cap).join(" ");
  const rows = Object.entries(byCourse).map(([course, ps]) => {
    const mission = ps.find(p => p.file.name === "MISSION");
    if (!mission) return null; // só cursos v3 (com MISSION) — exclui NotebookLM etc.
    // lessons concluídas = learning-records (.md indexável); o .html de lessons/ o Dataview não vê
    const lessons = ps.filter(p => (p.file.folder || "").includes("/learning-records")).length;
    const last = ps.reduce((m, p) => Math.max(m, p.file.mtime.toMillis()), 0);
    const type = mission.type ? ` · ${mission.type}` : "";
    return { course, mission, lessons, last, type };
  }).filter(Boolean).sort((a, b) => b.last - a.last);
  const DT = dv.luxon.DateTime;
  const card = r => `<a class="hp-card internal-link" href="${r.mission.file.path}" data-href="${r.mission.file.path}" style="--ac:#94e2d5">
    <span class="ico">🎓</span><span class="ttl">${pretty(r.course)}</span>
    <span class="meta">${r.lessons} ${r.lessons === 1 ? "lesson" : "lessons"}${r.type} · ${DT.fromMillis(r.last).toFormat("dd/MM")}</span></a>`;
  const c = dv.container.createDiv();
  c.innerHTML = rows.length
    ? `<div class="hp-label">Estudando agora · <span class="count">${rows.length}</span></div><div class="hp-grid">${rows.map(card).join("")}</div>`
    : `<div class="hp-label">Estudando agora</div><div class="hp-quote" style="text-align:left">Nenhum curso v3 ainda — roda /study-v3 pra iniciar um.</div>`;
} catch(e) { dv.paragraph("⚠️ estudando: " + e.message); }
```

```dataviewjs
// ===== Lendo agora =====
try {
  const reading = dv.pages("#book").where(b => b.status === "reading").array();
  const cover = b => b.cover
    ? `<div class="hp-cover"><img src="${b.cover}"></div>`
    : `<div class="hp-cover">${b.file.name}</div>`;
  const book = b => `<a class="hp-book internal-link" href="${b.file.name}" data-href="${b.file.path}">
    ${cover(b)}<div class="b-title">${b.file.name}</div><div class="b-author">${b.author ?? ""}</div></a>`;
  const c = dv.container.createDiv();
  c.innerHTML = `<div class="hp-label">Lendo agora · <span class="count">${reading.length}</span></div>`
    + (reading.length ? `<div class="hp-shelf">${reading.map(book).join("")}</div>` : `<div class="hp-quote" style="text-align:left">Nada em leitura — bora pegar um da estante 👇</div>`);
} catch(e) { dv.paragraph("⚠️ lendo: " + e.message); }
```

```dataviewjs
// ===== Lidos (ordena por data de fim, mais recente → antigo; vazios no fim) =====
try {
  const fin = b => (b.finished && b.finished.toMillis) ? b.finished.toMillis() : 0;
  const read = dv.pages("#book").where(b => b.status === "read").array()
    .sort((a, b) => fin(b) - fin(a));
  const cover = b => b.cover
    ? `<div class="hp-cover"><img src="${b.cover}"></div>`
    : `<div class="hp-cover">${b.file.name}</div>`;
  const stars = b => {
    const r = Number(b.rating) || 0;
    return r ? `<div class="b-rating">${"★".repeat(r)}${"☆".repeat(5 - r)}</div>` : "";
  };
  const when = b => (b.finished && b.finished.toFormat) ? `<div class="b-fin">${b.finished.toFormat("dd/MM/yy")}</div>` : "";
  const book = b => `<a class="hp-book internal-link" href="${b.file.name}" data-href="${b.file.path}">
    ${cover(b)}<div class="b-title">${b.file.name}</div><div class="b-author">${b.author ?? ""}</div>
    ${stars(b)}${when(b)}</a>`;
  const c = dv.container.createDiv();
  c.innerHTML = `<div class="hp-label">Lidos · <span class="count">${read.length}</span> livros</div>`
    + `<div class="hp-shelf">${read.map(book).join("")}</div>`;
} catch(e) { dv.paragraph("⚠️ lidos: " + e.message); }
```

```dataviewjs
// ===== Por ler (TBR) =====
try {
  const tbr = dv.pages("#book").where(b => b.status === "to-read").array()
    .sort((a, b) => dv.compare(a.file.name, b.file.name));
  const mini = b => b.cover ? `<span class="mini"><img src="${b.cover}"></span>` : `<span class="mini"></span>`;
  const row = b => `<a class="internal-link" href="${b.file.name}" data-href="${b.file.path}">
    ${mini(b)}<span><span class="t">${b.file.name}</span><br><span class="a">${b.author ?? ""}</span></span></a>`;
  const c = dv.container.createDiv();
  c.innerHTML = `<div class="hp-label">Por ler · <span class="count">${tbr.length}</span> na fila</div><div class="hp-tbr">${tbr.map(row).join("")}</div>`;
} catch(e) { dv.paragraph("⚠️ tbr: " + e.message); }
```

```dataviewjs
// ===== Últimos artigos =====
try {
  const arts = dv.pages('"2. Source Materials/Articles"').array()
    .sort((a, b) => b.file.mtime.toMillis() - a.file.mtime.toMillis()).slice(0, 6);
  const li = p => `<li><a class="internal-link" href="${p.file.name}" data-href="${p.file.path}">${p.file.name}</a>`
    + `<span class="when">${p.file.mtime.toFormat("dd/MM")}</span></li>`;
  const c = dv.container.createDiv();
  c.innerHTML = `<div class="hp-label">Últimos artigos</div><ul class="hp-arts">${arts.map(li).join("")}</ul>`;
} catch(e) { dv.paragraph("⚠️ artigos: " + e.message); }
```

```dataviewjs
// ===== Neste dia (flashback do daily) =====
try {
  const DT = dv.luxon.DateTime;
  const today = DT.now().startOf("day");
  const targets = [["1 semana", today.minus({ weeks: 1 })], ["1 mês", today.minus({ months: 1 })], ["1 ano", today.minus({ years: 1 })]];
  const rows = targets
    .map(([label, d]) => ({ label, d, p: dv.page(`5. Daily Journal/${d.toISODate()}.md`) }))
    .filter(r => r.p);
  const li = r => `<li><span class="when">há ${r.label}</span><a class="internal-link" href="${r.p.file.name}" data-href="${r.p.file.path}">${r.d.toFormat("cccc, dd/MM/yy")}</a></li>`;
  const c = dv.container.createDiv();
  if (rows.length) c.innerHTML = `<div class="hp-label">Neste dia</div><ul class="hp-arts">${rows.map(li).join("")}</ul>`;
} catch(e) { dv.paragraph("⚠️ neste dia: " + e.message); }
```

```dataviewjs
// ===== Footer =====
try {
  const links = [
    ["📓 Daily de hoje", dv.luxon.DateTime.now().toISODate()],
    ["📥 Study Queue", "Study Queue"],
    ["🧠 Mental Inventory", "Mental Inventory"],
    ["📚 Books (base)", "Books"],
  ];
  const c = dv.container.createDiv();
  c.innerHTML = `<div class="hp-foot">` + links.map(([t, h]) =>
    `<a class="internal-link" href="${h}" data-href="${h}">${t} →</a>`).join("") + `</div>`;
} catch(e) { dv.paragraph("⚠️ footer: " + e.message); }
```
