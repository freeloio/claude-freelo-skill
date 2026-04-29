# Changelog

Všechny významné změny v tomto skillu budou dokumentovány zde.

Formát vychází z [Keep a Changelog](https://keepachangelog.com/cs/1.1.0/),
verzování dle [Semantic Versioning](https://semver.org/lang/cs/).

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
