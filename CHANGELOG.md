# Changelog

Všechny významné změny v tomto skillu budou dokumentovány zde.

Formát vychází z [Keep a Changelog](https://keepachangelog.com/cs/1.1.0/),
verzování dle [Semantic Versioning](https://semver.org/lang/cs/).

## [Nevydáno]

### Přidáno

- **Reference `list-task-label-colors.md`** pro nový veřejný endpoint `GET /task-label-colors`, který vrací aktuální paletu barev task labelů (10 barev; `color` / `display_name` / `is_default`). Umožňuje Claudovi zjistit platné barvy živě místo hádání z pevného seznamu. Zaregistrováno v `SKILL.md` (sekce Labels) a propojeno ze sekce Colors v `references/curl-conventions.md`.

## [1.1.0] — 2026-06-03

### Změněno

- **Skill refaktorován na progressive disclosure.** Původní 1717-řádková
  SKILL.md byla rozdělena na štíhlou SKILL.md (~190 řádků, slouží jako
  router) a 100 souborů v `references/`, jeden na use case. Claude tak
  načítá jen tu část dokumentace, kterou potřebuje pro aktuální dotaz,
  místo aby měl v kontextu celou referenci. SKILL.md teď obsahuje pouze
  autentizaci, URL šablony pro odkazy, kritické formátovací pravidlo,
  shrnutí konvencí a tabulku referencí s trigger frázemi. Detailní
  endpointy, error handling, pitfalls a tipy se načítají z příslušného
  reference souboru. Žádné API změny — pure relokace.
  - **Úspora tokenů**: 88 % pro konverzaci s 1 operací, 82 % pro 5
    operací, 77 % pro 10 operací, 65 % i pro extrémní 20+ operací.
  - **Žádná změna v chování pro uživatele** — `/plugin install`
    příkaz stejný, skill řeší ty samé operace, jen vnitřní struktura
    je nová.
- **`references/curl-conventions.md`** — sdílený soubor s API basics,
  response shapes, error patterns, paginací, datovými formáty, currency,
  ID typy, prioritou, color whitelist, HTTP kódy, ~35 pitfalls a tipy.
  Načte se při psaní jakéhokoliv nového curl volání nebo dekódování chyby.
- **`references/response-formatting.md`** — plné rules pro formátování
  odkazů včetně bad/good příkladů a terminálového fallbacku. Načte se
  při skládání odpovědi zmiňující Freelo entitu. **Kritické pravidlo
  `[name](url)` z v1.0.1** zůstává prominentně nahoře v SKILL.md jako
  top-of-doc callout (nezregredováno).

### Opraveno

Čtyři menší doc gaps surfacované 14-UC subagent simulací PR review
proti MCP launch playbook scénářům:

- **`list-work-reports.md`** — doplněna kompletní tabulka query
  filterů včetně `date_reported_range[date_from]` /
  `date_reported_range[date_to]`. Server očekává **bracket form**
  (flat `date_reported_from` se silently ignoruje — ověřeno živě
  proti `api.freelo.io`). Plus canonical pattern pro týdenní team
  report a "use `minutes`, ne `time_spent`" guidance.
- **`invoices-create-not-supported.md`** (nový stub) — Freelo API
  nemá `POST /issued-invoice` endpoint. Stub matchuje pattern
  existujících "not supported" referencí
  (`edit-or-delete-tasklist-not-supported.md`,
  `delete-comment-not-supported.md`) — guide-uje model, aby spočítal
  totals z work reportů, pak uživatele poslal do Freelo UI vystavit
  fakturu, a po vystavení použil `mark-invoice-as-invoiced.md` pro
  uzavření smyčky.
- **`add-task-label-to-task.md`** — `GET /task-labels` endpoint
  neexistuje (vrací 404), `/project-labels/find-available` vrací jen
  project labels. Doplněn "Resolving a label UUID" oddíl s praktickým
  postupem (jq query nad `task.labels[]`) plus explicit "Bulk
  operations — never loop with `{name: ...}`" callout, aby model
  silently nevytvářel duplicitní šedé štítky při hromadných
  operacích.
- **`create-task.md`** — explicitně vyjmenovány validní `priority_enum`
  hodnoty (`l` / `m` / `h`), aby model nemusel pro tento jeden fakt
  loadovat `curl-conventions.md`.

### Telemetrie

- User-Agent `Freelo-Claude-Skill/1.0.1` → `1.1.0` ve všech 119
  curl příkladech napříč SKILL.md + 94 reference soubory, aby backend
  mohl rozlišit adopci nové verze.

## [1.0.1] — 2026-04-29

### Opraveno

- **Klikatelné odkazy na entity Freela** — Claude občas vypisoval ID/URL
  úkolů, projektů a dalších entit jako holé `https://app.freelo.io/...`
  místo `[Název](url)`. V chat appce uživatel viděl URL, v terminálu
  taky URL bez názvu — v obou případech musel klikat naslepo, aby zjistil,
  o jaký úkol jde. Pravidlo "always link entities" už ve skillu bylo, ale
  bylo zakopané hluboko v dokumentu a chybělo mu modelování antipatternu
  v tabulkách (což byl nejčastější failure mode).
- **Fallback pro terminálový renderer** — Claude Code v terminálu (TUI)
  občas collapsne `[name](url)` markdown na samotnou URL v table cellách.
  Skill teď instruuje Claudovi: po upozornění uživatele ("vidím jen URL",
  "v terminálu", "v CLI") přepnout na dvouřádkový plain-text formát
  (název na jednom řádku, URL pod ním) — ten funguje v každém rendereru
  a uživatel tak vidí název i klikatelný odkaz.

### Změněno

- **Top-of-skill callout** — kritické pravidlo o `[name](url)` formátu je
  nově hned po `API reference` linku, aby ho Claude nepřehlédl.
- **Příklady v sekci "Response formatting"** — přidány konkrétní
  bad/good příklady pro tabulky (přesně modeluje failure mode, který se
  v praxi reálně vyskytl) a anti-pattern bare URL v bullet listu.
- **Pravidla zpřísněna** — link text MUSÍ být název entity, NIKDY URL.
  Fallback `Task #{id}` jen když se název opravdu nepodařilo načíst,
  ne jako shortcut.

### Telemetrie

- User-Agent `Freelo-Claude-Skill/1.0.0` → `1.0.1` ve všech příkladech
  v SKILL.md, aby backend mohl rozlišit adopci nové verze.

## [1.0.0] — 2026-04-15

První veřejné vydání. Claude Code skill pro ovládání Freelo.io přirozeným
jazykem — projekty, úkoly, time tracking, soubory, štítky, custom fields
a další.
