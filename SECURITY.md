# Bezpečnostní zásady

## 🔐 Hlášení bezpečnostních problémů

Pokud jste objevili bezpečnostní zranitelnost v tomto skillu nebo způsob, jakým komunikuje s Freelo API, **nepublikujte ji veřejně v GitHub Issues**.

Místo toho prosím napište na **info@freelo.io** s předmětem `[SECURITY] claude-freelo-skill: <stručný popis>`.

Do zprávy zahrňte:

1. **Popis problému** — co je zranitelnost a jaký je její dopad
2. **Reprodukce** — kroky, jak ji vyvolat
3. **Verzi skillu** — z `plugin.json`
4. **Návrh řešení** — pokud máte
5. **Vaše kontaktní údaje** — pro follow-up

Snažíme se odpovědět **do 5 pracovních dní** a publikovat opravu co nejdříve.

## 🛡️ Best practices pro uživatele

Skill přistupuje k Freelo API **vaším jménem** přes API klíč. Z hlediska bezpečnosti:

### API klíč

- ❌ **Nikdy neukládejte API klíč** do veřejných repositorů (`git` → `.gitignore`)
- ❌ **Nesdílejte API klíč** s kolegy — každý ať si vygeneruje vlastní
- ❌ **Nepoužívejte na sdílených počítačích**
- ✅ **Ukládejte do `~/.claude/settings.json`** (lokální, ne v gitu)
- ✅ **Při podezření na únik klíč okamžitě obnovte** ve Freelo nastavení

### Destruktivní akce

Skill umožňuje Claude Code provádět nevratné operace:

- Mazání úkolů
- Mazání projektů
- Archivace
- Hromadné přesuny

**Před produkčním nasazením doporučujeme:**

1. Vyzkoušet workflow na **testovacím projektu**
2. **Číst, co Claude navrhuje** před potvrzením destruktivní akce
3. Mít **zálohu důležitých dat** mimo Freelo

### Citlivá data v promptech

Cokoli, co napíšete Claudovi, projde přes Anthropic API. Pokud Freelo úkoly obsahují citlivé informace (klientská data, smlouvy, finanční výkazy), pamatujte:

- Anthropic má vlastní [zásady ochrany soukromí](https://www.anthropic.com/legal/privacy)
- Pro maximální izolaci zvažte placený Claude pro Workgroups s kontrolou dat

## 🔒 Co skill **dělá** a **nedělá**

✅ **Dělá:**
- Posílá HTTP requesty na `https://api.freelo.io/v1/*`
- Autentizuje se přes HTTP Basic (váš `FREELO_EMAIL` + `FREELO_API_KEY`)
- Posílá `User-Agent: ClaudeCodeSkill` (žádné PII v hlavičce)
- Stahuje a nahrává soubory přes Freelo API

❌ **Nedělá:**
- Nikam neposílá vaše údaje mimo Freelo a Anthropic
- Nepřistupuje k jiným API než Freelo
- Neukládá data lokálně (kromě dočasných souborů, které smaže)
- Nepřidává si vlastní webhooky ani persistent connections

## 📜 Verze tohoto dokumentu

Aktualizace k 14. dubnu 2026 (pro v1.0.0).
