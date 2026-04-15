# Příspěvek do Freelo skill

Děkujeme za zájem o vylepšení tohoto skillu! 🎉

## 🐛 Reportování chyb

Pokud jste narazili na bug, otevřte [GitHub Issue](https://github.com/freeloio/claude-freelo-skill/issues/new?template=bug_report.md) s následujícími informacemi:

1. **Co jste chtěli udělat** — váš prompt nebo akce
2. **Co Claude udělal** — odpověď nebo chybová hláška
3. **Co jste očekávali**
4. **Verze skillu** — najdete v `plugin.json`
5. **HTTP status code** chybové odpovědi (pokud znáte)

## 💡 Návrhy na vylepšení

Máte nápad, co skill mohl umět lépe? Otevřte [Feature request issue](https://github.com/freeloio/claude-freelo-skill/issues/new?template=feature_request.md).

Nejcennější návrhy popisují:
- **Use case** — kdy by byla featura užitečná, příklad promptu
- **Persona** — kdo by ji používal (PM, vývojář, account manager…)
- **Frekvence** — jak často by se používala

## 🛠️ Pull requesty

Před otevřením PR doporučujeme:

1. **Otevřít issue** a probrat změnu — vyhneme se zbytečné práci, pokud směr neodpovídá.
2. **Otestovat změnu proti reálnému Freelo API** — ne jen lokálně. SKILL.md je živá dokumentace API.
3. **Aktualizovat dokumentaci** v relevantních sekcích SKILL.md.
4. **Aktualizovat CHANGELOG.md** — sekce Unreleased.

### Co je v PR vítané

- Oprava chyb v dokumentaci (nesprávné endpointy, body shapes, error messages)
- Nové scénáře a workflow examples v Tips sekci
- Nové troubleshooting body z reálných případů
- Aktualizace pokud Freelo API přidá nové endpointy nebo změní existující

### Co PR vyžaduje

- **Skutečné API ověření** — popis přesných curl příkazů a očekávaných odpovědí, ne spekulace
- **Záchovu zpětné kompatibility** — nebreakovat fungující workflows
- **Češtinu pro komunikaci** + angličtinu pro samotný SKILL.md (Claude pracuje lépe v EN)

## 🏗️ Lokální testování

Skill je čistě dokumentace — nejedná se o spustitelný kód. Test = vyzkoušet promot v Claude Code se zkušebním projektem ve Freelu.

Doporučená flow:
1. Vytvořte si prázdný testovací projekt ve Freelu
2. Nastavte `FREELO_EMAIL` a `FREELO_API_KEY` v `~/.claude/settings.json`
3. Restartujte Claude Code
4. Použijte testovací prompty z [README.md](README.md#-příklady-použití)
5. Verify přímo ve Freelu, že se akce skutečně provedla správně

## 📞 Otázky

- 💬 **Diskuze a otázky** → [GitHub Discussions](https://github.com/freeloio/claude-freelo-skill/discussions) (pokud zapnuté) nebo Issues
- 📧 **Důvěrné dotazy** → info@freelo.io

## 📜 Licence

Příspěvkem souhlasíte, že váš kód bude licencován pod [MIT licencí](LICENSE).
