---
name: arc42-docs
description: Erstellt und pflegt Architekturdokumentation nach arc42. Verwenden, wenn arc42, Architekturdoku, Architekturdokumentation, Bausteinsicht, Laufzeitsicht, Verteilungssicht, Kontextabgrenzung, Qualitätsszenarien oder ADR (Architecture Decision Record) erwähnt werden oder eine bestehende Architekturdoku aktualisiert werden soll.
---

# arc42-Architekturdokumentation

Dieser Skill unterstützt beim Erstellen, Ergänzen und Reviewen von Architekturdokumentation nach dem arc42-Template. Die Dokumentation liegt als Markdown im Repository (Docs-as-Code) und wird zusammen mit dem Code versioniert und per Pull Request reviewt.

Die vollständige Kapitelstruktur mit Leitfragen pro Kapitel steht in [references/arc42-template.md](references/arc42-template.md).

## Wann anwenden

- Ein neues System oder ein neuer Service braucht eine Architekturdokumentation.
- Eine bestehende arc42-Doku muss nach einer Änderung (neuer Baustein, neue Schnittstelle, neue Infrastruktur) nachgeführt werden.
- Eine Architekturentscheidung soll als ADR festgehalten werden.
- Einzelne Sichten (Kontextabgrenzung, Bausteinsicht, Laufzeitsicht, Verteilungssicht) sollen erstellt oder überarbeitet werden.
- Eine bestehende Doku soll auf Vollständigkeit und Aktualität geprüft werden.

Nicht anwenden für reine API-Referenzen (dafür OpenAPI/AsyncAPI) oder für Betriebshandbücher.

## Vorgehen

### 1. Kontext klären

- Prüfe, ob bereits eine Doku existiert, typischerweise unter `docs/architecture/` oder `docs/arc42/`. Falls ja: bestehende Struktur übernehmen, nicht neu anlegen.
- Lies README, Build-Dateien, Deployment-Konfiguration (z. B. Helm-Charts, Kubernetes-Manifeste) und Schnittstellen-Definitionen, um Fakten statt Annahmen zu dokumentieren.
- Kläre mit der anfragenden Person Zielpublikum und Detaillierungsgrad. Im Zweifel: kompakt starten und die Kapitel 1, 3, 4, 5 und 9 priorisieren.

### 2. Struktur anlegen

Lege bei einer neuen Doku folgende Struktur an (eine Datei pro Kapitel, nummeriert):

```text
docs/architecture/
├── 01-einfuehrung-und-ziele.md
├── 02-randbedingungen.md
├── 03-kontextabgrenzung.md
├── 04-loesungsstrategie.md
├── 05-bausteinsicht.md
├── 06-laufzeitsicht.md
├── 07-verteilungssicht.md
├── 08-querschnittliche-konzepte.md
├── 09-architekturentscheidungen.md
├── 10-qualitaetsanforderungen.md
├── 11-risiken-und-technische-schulden.md
├── 12-glossar.md
└── adr/
    └── 0001-<kurztitel>.md
```

Kapitel, zu denen noch nichts Belastbares bekannt ist, erhalten einen kurzen Satz, was dort künftig dokumentiert wird, und wer dafür zuständig ist. Keine leeren Dateien.

### 3. Inhalte erarbeiten

- **Kontextabgrenzung (Kap. 3):** Fachlichen und technischen Kontext trennen. Jedes Nachbarsystem mit Kommunikationsart (REST, Kafka, Batch, …) und Richtung aufführen.
- **Bausteinsicht (Kap. 5):** Top-down von Ebene 1 (Whitebox Gesamtsystem) zu Ebene 2. Jeder Baustein bekommt Verantwortung, Schnittstellen und Ablageort im Code.
- **Laufzeitsicht (Kap. 6):** Nur die zwei bis fünf architekturrelevanten Szenarien beschreiben, nicht jeden Use Case.
- **Diagramme:** Als Code (Mermaid oder PlantUML) direkt im Markdown, damit sie im Review diffbar sind. Kein Diagramm ohne Legende oder erklärenden Text.

### 4. ADR schreiben

Jede Entscheidung mit Tragweite über ein einzelnes Team oder einen Sprint hinaus wird als ADR unter `adr/` festgehalten. Format:

```markdown
# ADR-0001: <Entscheidung als kurzer Satz>

- Status: vorgeschlagen | akzeptiert | ersetzt durch ADR-XXXX | verworfen
- Datum: JJJJ-MM-TT
- Entscheidende: <Rollen oder Teams>

## Kontext
Welches Problem, welche Kräfte und Randbedingungen?

## Entscheidung
Was wurde entschieden, formuliert im Aktiv («Wir verwenden …»)?

## Betrachtete Alternativen
Optionen mit Vor- und Nachteilen.

## Konsequenzen
Positive und negative Folgen, Folgearbeiten.
```

ADRs werden nie gelöscht. Überholte ADRs erhalten den Status «ersetzt durch» und einen Link auf die Nachfolge-ADR. Kapitel 9 enthält eine Tabelle aller ADRs mit Status.

### 5. Abschliessen

- Verlinkungen zwischen Kapiteln und ADRs prüfen.
- Glossar mit allen Fachbegriffen ergänzen, die im Text vorkommen.
- Änderungen in einem eigenen Pull Request einreichen; im PR-Text die betroffenen Kapitel nennen.

## Qualitätskriterien

- **Faktentreue:** Jede Aussage ist im Code, in der Konfiguration oder durch eine Entscheidung belegbar. Annahmen werden als solche markiert.
- **Aktualität:** Die Doku beschreibt den aktuellen Stand im Haupt-Branch, nicht einen Wunschzustand. Zielarchitektur wird explizit als solche gekennzeichnet.
- **Nachvollziehbarkeit:** Wesentliche Entscheidungen sind als ADR mit Begründung und Alternativen dokumentiert.
- **Konsistenz:** Bausteinnamen sind in allen Sichten, Diagrammen und im Code identisch.
- **Messbare Qualitätsziele:** Qualitätsanforderungen in Kapitel 10 sind als Szenarien mit Stimulus, Umgebung und messbarer Antwort formuliert (z. B. «95 % der Anfragen unter 300 ms bei 200 Requests/s»).
- **Angemessener Umfang:** So knapp wie möglich, so ausführlich wie nötig. Lieber ein präzises Diagramm mit fünf Sätzen als drei Seiten Fliesstext.
- **Sprache:** Deutsch in Schweizer Schreibweise (ss statt ß), Tech-Begriffe auf Englisch.
