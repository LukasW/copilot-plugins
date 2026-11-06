# CSS Copilot Plugins

Interner Plugin-Marketplace `css-plugins` für GitHub Copilot CLI und Claude Code.

Repository: https://scm.grudligstrasse.ch/css/copilot-plugins

## Zweck

Dieser Marketplace bündelt Plugins mit Skills, die unsere Arbeitsweise und Standards abbilden. So arbeiten Coding-Agents in allen Teams nach denselben Konventionen, etwa beim Schreiben von Architekturdokumentation oder beim Design von Kafka-Events. Die Plugins folgen dem Format Agent Plugins 1.0.

Aktuell enthaltene Plugins:

| Plugin         | Version | Skills                                  |
|----------------|---------|-----------------------------------------|
| `css-baseline` | 0.1.0   | `arc42-docs`, `kafka-event-design`      |

## Installation

Das Repository liegt auf Gitea und ist privat. Deshalb muss die volle Git-URL angegeben werden; die Kurzform `owner/repo` funktioniert nur für GitHub. Git braucht Lesezugriff auf das Repository (HTTPS mit Credential Helper oder SSH via `ssh://git@scm.grudligstrasse.ch/css/copilot-plugins.git`).

### GitHub Copilot CLI

```bash
copilot plugin marketplace add https://scm.grudligstrasse.ch/css/copilot-plugins.git
copilot plugin install css-baseline@css-plugins
```

Verfügbare Plugins anzeigen:

```bash
copilot plugin marketplace browse css-plugins
```

### Claude Code

```bash
/plugin marketplace add https://scm.grudligstrasse.ch/css/copilot-plugins.git
/plugin install css-baseline@css-plugins
```

## Struktur

```text
.
├── .github/plugin/marketplace.json   # Marketplace-Definition für Copilot CLI
├── .claude-plugin/marketplace.json   # identische Definition für Claude Code
└── plugins/
    └── <plugin-name>/
        ├── plugin.json
        └── skills/
            └── <skill-name>/
                ├── SKILL.md
                └── references/       # optionale Detail-Dokumente
```

## Neues Plugin oder Skill hinzufügen

### Neuen Skill in einem bestehenden Plugin

1. Ordner `plugins/<plugin>/skills/<skill-name>/` anlegen. Der Ordnername ist lowercase mit Bindestrichen und entspricht exakt dem `name` im Frontmatter.
2. `SKILL.md` mit Frontmatter anlegen:

   ```markdown
   ---
   name: <skill-name>
   description: Was der Skill tut und wann er verwendet wird, inklusive der Trigger-Begriffe.
   ---
   ```

3. Body mit den Abschnitten «Wann anwenden», «Vorgehen» und «Qualitätskriterien» schreiben. Die `SKILL.md` bleibt unter 200 Zeilen; Details gehören nach `references/` und werden aus der `SKILL.md` verlinkt.
4. `version` in `plugins/<plugin>/plugin.json` und in beiden `marketplace.json` erhöhen (Semantic Versioning: neuer Skill = Minor, Korrektur = Patch).

### Neues Plugin

1. Ordner `plugins/<plugin-name>/` mit `plugin.json` anlegen:

   ```json
   {
     "$schema": "https://agent-plugins.org/schemas/1.0.0/plugin.schema.json",
     "name": "<plugin-name>",
     "description": "Kurze Beschreibung",
     "version": "0.1.0",
     "author": { "name": "<Team-Name>", "email": "team@css.ch" }
   }
   ```

2. Skills wie oben beschrieben unter `skills/` anlegen.
3. Das Plugin in **beiden** Dateien `.github/plugin/marketplace.json` und `.claude-plugin/marketplace.json` in der Liste `plugins` eintragen. Der `source`-Pfad ist relativ zum Repo-Root, z. B. `./plugins/<plugin-name>`. Die beiden Dateien müssen identisch bleiben.

### Lokal testen

```bash
# im Repo-Root ausführen; absoluter Pfad nötig, ein relativer Pfad wird als GitHub-Repo interpretiert
copilot plugin marketplace add "$PWD"
copilot plugin marketplace browse css-plugins
copilot plugin install <plugin-name>@css-plugins
# nach dem Test wieder entfernen
copilot plugin uninstall <plugin-name>@css-plugins
copilot plugin marketplace remove css-plugins
```

## Sprache und Stil

- Doku- und Skill-Texte auf Deutsch in Schweizer Schreibweise (ss statt ß).
- Tech-Begriffe bleiben Englisch (z. B. Pull Request, Topic, Schema Registry).
- Keine Platzhalter-Texte wie «TODO» oder «Lorem ipsum» in Skills.

## Review-Prozess

- Direkte Pushes auf `main` sind nicht erlaubt. Jede Änderung erfolgt über einen Pull Request in Gitea.
- Jeder Pull Request braucht mindestens ein Approval der in `CODEOWNERS` hinterlegten Owner (Team `css/Owners`). Gitea setzt diese automatisch als Reviewer.
- Durchgesetzt wird das über die Branch Protection von `main` (Push nur via PR, mindestens ein Approval, Approval durch Code Owner erforderlich).
- Im PR-Text beschreiben: welches Plugin bzw. welcher Skill, was geändert wurde und wie lokal getestet wurde.
- Reviewende prüfen insbesondere:
  - Ordnername und `name` im Frontmatter stimmen überein.
  - Die `description` enthält die relevanten Trigger-Begriffe.
  - Beide `marketplace.json` sind identisch und die Versionen sind erhöht.
  - Inhalte sind fachlich korrekt und entsprechen unseren Standards.
