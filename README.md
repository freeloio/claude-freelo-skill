# Freelo skill pro Claude

Ovládejte [Freelo.io](https://freelo.io) přes Claude přirozeným jazykem. Místo klikání v UI stačí Claudovi napsat, co potřebujete — a on to udělá za vás.

> *"Vytvoř úkol pro Petra 'Připravit nabídku pro klienta XYZ', deadline pátek, priorita vysoká"*
> *"Co mám dneska na programu? Co je po termínu?"*
> *"Tady je zápis z porady — vytvoř z toho úkoly ve Freelu."*

## ✨ Co skill umí

Po instalaci Claude umí ve Freelu samostatně:

- 📋 **Úkoly** — vytvářet, upravovat, dokončovat, přesouvat, mazat (včetně podúkolů, komentářů, popisů a štítků)
- 📁 **Projekty a tasklisty** — listovat, zakládat, archivovat, zakládat ze šablon
- ⏱️ **Time tracking** — spustit/zastavit timer, vytvářet ruční work reporty, generovat časové přehledy
- 🔖 **Štítky** — projektové i úkolové, hromadné přiřazování
- 📎 **Soubory** — upload, připojení k úkolům/komentářům, stažení
- 📝 **Poznámky** s HTML obsahem
- 🔍 **Vyhledávání** napříč úkoly, projekty, komentáři
- 👥 **Workers** — pozvánky, odebrání, přehled vytížení
- 🏖️ **Out of Office** — zapnutí, vypnutí, kontrola stavu
- 📨 **Notifikace** a aktivita (audit log)
- 🧾 **Faktury** — listing, označení jako fakturováno
- ⚙️ **Custom fields** — text, číslo, datum, výčet, link a další

## 🚀 Instalace

Skill je určený pro **Claude Code** (terminál nebo desktop app v **Code** režimu). V Claude.ai web chatu nefunguje spolehlivě — pro REST API volání potřebuje přístup k proměnným prostředí, a ty se v čistém chatu nedají uložit napříč session.

Instalace je ve **3 krocích**: nainstaluj skill → přidej credentials → restartuj Claude Code.

### Krok 1. Nainstaluj skill

V Claude Code spusťte **dva příkazy po sobě** (každý zvlášť jako samostatný prompt — nespojovat do jednoho):

**1a.** Přidej marketplace:

```
/plugin marketplace add freeloio/claude-freelo-skill
```

Počkej na potvrzení (typicky "Marketplace added"), pak:

**1b.** Nainstaluj skill:

```
/plugin install freelo@claude-freelo-skill
```

> ⚠️ **Nekopíruj oba příkazy najednou** — Claude Code se slash-příkazy (začínající `/`) musí zpracovávat jeden po druhém. Pokud vložíš oba řádky naráz, interpretuje druhý jako argument prvního a dostaneš chybu *"URL rejected: Malformed input"*.

### Krok 2. Přidej své Freelo credentials do `settings.json`

Claude Code má globální konfigurační soubor `~/.claude/settings.json`. Pokud jsi s tím nikdy nepracoval/a, nejjednodušší cesta ho otevřít je **jeden příkaz v terminálu**:

**macOS** — otevře soubor v TextEditu (pokud neexistuje, TextEdit se zeptá jestli ho má vytvořit — řekni **Ano**):
```bash
open -e ~/.claude/settings.json
```

**Windows** — otevře v Notepadu:
```
notepad %USERPROFILE%\.claude\settings.json
```

**Linux** — otevře v editoru `nano`:
```
nano ~/.claude/settings.json
```

#### Alternativně — přes Finder / File Explorer

Pokud terminál nepoužíváš:

- **macOS Finder**: `Cmd + Shift + G` → zadej `~/.claude` → Enter → najdi (nebo vytvoř) `settings.json`
- **Windows File Explorer**: do adresního řádku zadej `%USERPROFILE%\.claude\` → Enter → najdi (nebo vytvoř) `settings.json`

Složka `.claude` je **skrytá** (začíná tečkou), ale cesta výše ji najde bez problému.

#### Co do souboru napsat

**Pokud je soubor prázdný** (právě jsi ho vytvořil), vlož celý tenhle obsah a doplň své údaje:

```json
{
  "env": {
    "FREELO_EMAIL": "tvuj@email.cz",
    "FREELO_API_KEY": "tvuj-api-klic-z-freela"
  }
}
```

**Pokud už v něm něco je** (třeba `enabledPlugins` nebo jiná nastavení), přidej sekci `env` dovnitř složených závorek. Výsledek má vypadat takto:

```json
{
  "enabledPlugins": { ... },
  "effortLevel": "max",
  "env": {
    "FREELO_EMAIL": "tvuj@email.cz",
    "FREELO_API_KEY": "tvuj-api-klic-z-freela"
  }
}
```

> ⚠️ **Pozor na čárky** — mezi jednotlivými sekcemi musí být čárka (`,`), za poslední sekcí ne. Nejčastější chyba začátečníků v JSONu.

> 🔒 **`settings.json` nikdy nenahrávej do gitu.** Obsahuje tvůj API klíč, který má plný přístup k účtu.

### Krok 3. Restart Claude Code

**Quit Claude Code úplně**, ne jen zavřít okno:

- **macOS**: `Cmd + Q` v menu aplikace (ne křížek na okně)
- **Windows / Linux**: File → Exit

Po znovuspuštění skill naběhne s tvými credentials a můžeš začít. Vyzkoušej třeba: *"Jaké úkoly mám na dnešek ve Freelu?"*

## 🔑 Získání Freelo API klíče

Najdete v nastavení Freela: **Profil → API klíč** ([nápověda](https://www.freelo.io/cs/napoveda)).

Klíč je **43-znakový řetězec**, něco jako `xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`. Celý ho zkopíruj a vlož do `settings.json`.

> 🔐 **Nepoužívejte API klíč na sdílených počítačích.** Má plný přístup k účtu. Pokud máš podezření že unikl (screenshot, omylem sdílený v chatu, atd.), okamžitě vygeneruj nový — starý přestane platit.

## 💬 Příklady použití

### Správa úkolů

- *"Vytvoř úkol **Připravit prezentaci pro klienta XYZ** v projektu Marketing, přiřaď Petrovi, deadline pátek"*
- *"Jaké úkoly mám dnes? Co je po termínu?"*
- *"Přesuň nedokončené úkoly ze Sprintu 3 do Sprintu 4"*
- *"Označ úkol Nabídka ABC jako hotový a přidej komentář, že byla odeslána klientovi"*

### Řízení projektů

- *"Jak na tom je projekt Redesign webu? Kolik úkolů je hotových, kolik po termínu?"*
- *"Založ nový projekt pro klienta XYZ podle šablony Onboarding"*
- *"Kdo v týmu má nejvíc úkolů v projektu Marketing?"*

### Porady a komunikace

- *"Tady je zápis z dnešní porady: [text]. Vytvoř z toho úkoly ve Freelu."*
- *"Napiš ke úkolu Redesign komentář: wireframy hotové, čekáme na feedback"*
- *"Ulož zápis z dnešní porady jako poznámku k projektu Marketing Q1"*

### Time tracking

- *"Začni měřit čas na úkolu Nabídka pro XYZ"*
- *"Kolik hodin tento týden odpracoval tým na projektu Redesign?"*
- *"Zastav tracking a uložit"*

### Soubory

- *"Připoj tento PDF ke úkolu Návrh smlouvy"*
- *"Stáhni soubor [UUID] a ulož ho lokálně"*

## ⚠️ Důležité — než začnete

**Skill provádí reálné akce ve vašem účtu.** Vytváří, upravuje a maže data ve Freelu — a každá akce je nevratná. Před produkčním nasazením doporučujeme:

1. **Začít na testovacím projektu** — než pustíte skill na ostrá data, vyzkoušejte si workflow na zkušebním projektu.
2. **Vždy si ověřit, co Claude navrhuje** — před destruktivními akcemi (smazání, archivace) si přečtěte, co se chystá udělat. Claude by se měl ptát, ale potvrzení je vaše zodpovědnost.
3. **API klíč nesdílejte** — má plný přístup k účtu.

## 🛡️ Co skill NEumí (známé limitace)

- ❌ **Mazat tasklisty** — Freelo API nepodporuje (je nutné v UI)
- ❌ **Mazat komentáře** — API nepodporuje (lze pouze upravit obsah)
- ❌ **Připojovat soubory k poznámkám** — API tichý drop (úkolům + popisům úkolů ano)
- ❌ **Vytvářet podúkoly podúkolů** (nested) — API vrací nepoužitelný výsledek; doporučujeme úkoly zploštit
- ⚠️ **Task estimates a public links** — placené featury (HTTP 402 pokud váš plán nezahrnuje)

Tyto limitace jsou důsledek omezení Freelo API — skill je dobře dokumentuje a Claude se s nimi umí elegantně vypořádat.

## 🔧 Co skill používá pod kapotou

- **Freelo REST API** — autentizace HTTP Basic (email + API klíč)
- **Rate limit** — 25 requestů za minutu (skill toto respektuje)
- **Plný HTML** v komentářích a popisech (s automatickou XSS sanitizací na straně Freela)
- **Plnou Czech/Unicode** podporu (diakritika, emoji)
- **User-Agent: `Freelo-Claude-Skill/{version}`** — každý request posílá tuto hlavičku, aby Freelo backend mohl agregovat usage statistiky skillu (unikátní uživatelé, nejpoužívanější endpointy, version adoption). Neobsahuje žádná osobní data — jen verzi skillu.

Plnou API referenci najdete na [api.freelo.io/docs/v1/freelo-api](https://api.freelo.io/docs/v1/freelo-api).

## 🛠️ Nefunguje to? (časté problémy)

### *"URL rejected: Malformed input to a URL function"* při instalaci

Kopírujete oba instalační příkazy (`/plugin marketplace add …` a `/plugin install …`) najednou.
→ Zadejte je **postupně** jako samostatné prompty. Nejdřív počkejte na potvrzení prvního, pak spusťte druhý.

### Skill se načte, ale hlásí *"credentials aren't set as environment variables"*

Nejspíš jsi nerestartoval Claude Code úplně — jen zavřel okno.
→ Dej **Quit** (`Cmd + Q` na macOS, File → Exit jinde) a spusť znovu.

### Vidíš prázdný účet, ale úkoly jsou tam

Freelo má pro nové účty **"onboarding" demo projekt** ("Vyzkoušej si Freelo"), který se nezobrazuje v listingu přes API. Pokud tě napadne, kde jsou úkoly, řekni Claudu přímo ID nebo název projektu:

> *"Moje úkoly jsou v projektu 'Vyzkoušej si Freelo'. Mrkni tam přímo."*

### Skill se přihlašuje pod jinou identitou než tvou

Pokud Claude v odpovědi uvádí email/jméno, které neznáš — máš v `settings.json` API klíč z jiného Freelo účtu (třeba ze sdíleného zkušebního).
→ Vygeneruj klíč v **tvém** účtu (Profil → API klíč) a přepiš `FREELO_EMAIL` **i** `FREELO_API_KEY` v `settings.json`. Oba musí patřit stejnému účtu.

### TextEdit odmítá uložit `.json`

TextEdit defaultně ukládá v Rich Text formátu, který JSON parser nepřečte.
→ V menu **Format → Make Plain Text** (nebo `Cmd + Shift + T`), pak ulož. Nebo použij VS Code či jiný programátorský editor.

### JSON syntax error po uložení

Nejčastější chyba — chybí/přebývá čárka mezi sekcemi, nebo chybí uvozovky.
→ Vlož obsah sem: [jsonlint.com](https://jsonlint.com) — ukáže přesně kde je chyba.

### Claude hlásí HTTP 402 Payment Required

Pokoušíš se o funkci, která vyžaduje **placený Freelo plán** (task estimates, task public links, některé custom field typy). Chyba přichází z Freelo API — skill na ní nemůže nic změnit.

### `"HTTP Authorization header was not found"`

API klíč se nedostává do requestu. Zkontroluj:
- `settings.json` má proměnné přesně jako `FREELO_EMAIL` a `FREELO_API_KEY` (případ na pozor — žádné pomlčky, žádný překlep)
- Hodnoty jsou v uvozovkách, neprázdné
- Po úpravě jsi restartoval Claude Code úplně (Quit)

### API klíč nevím kam dát / ztratil jsem ho

V Freelu: **Nastavení profilu → API klíč**. Pokud žádný nevidíš, klikni **Generovat**. Pokud ho ztratíš, generuj nový — starý okamžitě přestane fungovat.

Pokud ani tohle nepomůže, [otevři issue na GitHubu](https://github.com/freeloio/claude-freelo-skill/issues) s konkrétní chybovou hláškou, co ti skill vrátil, a prostředím (macOS/Windows + verze Claude Code).

## 🆘 Podpora

- 🐛 **Bug nebo žádost o feature** → [GitHub Issues](https://github.com/freeloio/claude-freelo-skill/issues)
- 📧 **Obecné dotazy** → info@freelo.io
- 📚 **Dokumentace Freela** → [www.freelo.io/cs/napoveda](https://www.freelo.io/cs/napoveda)

## 📜 License

[MIT](LICENSE) © 2026 Freelo
