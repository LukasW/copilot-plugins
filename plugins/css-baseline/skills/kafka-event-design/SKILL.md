---
name: kafka-event-design
description: Unterstützt beim Design von Kafka-Events, Topics und Avro-Schemas. Verwenden bei Kafka, Event-Design, Event-Naming, Topic-Naming, Avro, Schema Registry, Schema-Evolution, Event-Versionierung, Kompatibilität (BACKWARD, FORWARD, FULL) oder wenn ein neues Event bzw. Topic definiert oder ein bestehendes Schema geändert werden soll.
---

# Kafka-Event-Design

Dieser Skill hilft, Kafka-Events konsistent zu benennen, Avro-Schemas sauber zu modellieren und Schema-Änderungen kompatibel zu versionieren. Ziel sind Events, die für Consumer verständlich, stabil und ohne Absprache-Overhead weiterentwickelbar sind.

Die verbindlichen Namenskonventionen für Topics, Events, Felder und Namespaces stehen in [references/naming-conventions.md](references/naming-conventions.md).

## Wann anwenden

- Ein neues Event oder ein neues Topic wird eingeführt.
- Ein bestehendes Avro-Schema soll geändert werden (Feld hinzufügen, umbenennen, Typ ändern, entfernen).
- Es muss entschieden werden, ob eine Änderung kompatibel ist oder eine neue Major-Version braucht.
- Ein Event-Design soll im Review geprüft werden.
- Es ist unklar, ob ein Event als Domain-Event, Change-Event (CDC) oder Command modelliert werden soll.

## Vorgehen

### 1. Fachlichkeit klären

- Was ist passiert? Ein Event beschreibt eine abgeschlossene fachliche Tatsache in der Vergangenheitsform (`PolicyCreated`, nicht `CreatePolicy`).
- Wer ist Owner des Events? Genau ein Producer-Team ist für Topic und Schema verantwortlich.
- Wer konsumiert das Event, und welche Informationen brauchen die Consumer wirklich?
- Event-Typ festlegen:
  - **Domain-Event:** fachliche Tatsache, schlank, enthält Schlüssel und relevante Änderungen. Standardfall.
  - **Event-carried State Transfer:** enthält den vollständigen fachlichen Zustand, wenn Consumer ohne Rückfrage arbeiten sollen.
  - **Command:** Aufforderung an ein bestimmtes System. Nur in Ausnahmefällen über Kafka; dann im Imperativ benennen und ein eigenes Topic verwenden.

### 2. Topic und Key festlegen

- Topic-Name gemäss Konvention `<domain>.<subdomain>.<entity>.<event-type>.v<major>` (Details in der Referenz).
- Message-Key ist die fachliche ID des Aggregats (z. B. `policyId`), damit alle Events zu einem Aggregat in derselben Partition und damit in der richtigen Reihenfolge landen.
- Partitionsanzahl, Retention und `cleanup.policy` (delete oder compact) bewusst wählen und begründen.

### 3. Avro-Schema modellieren

- Ein Schema pro Event-Typ, abgelegt im Repository des Producers unter `src/main/avro/`.
- Jedes Event enthält einen Header-Record mit Metadaten:

```json
{
  "type": "record",
  "name": "PolicyCreated",
  "namespace": "ch.css.policy.events.v1",
  "doc": "Eine neue Police wurde erfasst und ist gültig.",
  "fields": [
    { "name": "metadata", "type": "ch.css.common.events.EventMetadata", "doc": "Technische Metadaten des Events." },
    { "name": "policyId", "type": "string", "doc": "Fachliche ID der Police." },
    { "name": "validFrom", "type": { "type": "int", "logicalType": "date" }, "doc": "Gültigkeitsbeginn der Police." },
    { "name": "productCode", "type": "string", "doc": "Code des versicherten Produkts." },
    { "name": "channel", "type": ["null", "string"], "default": null, "doc": "Erfassungskanal, falls bekannt." }
  ]
}
```

- `EventMetadata` enthält mindestens `eventId` (UUID), `eventTime` (`timestamp-millis`), `source` (produzierendes System) und `correlationId`.
- Jedes Feld hat ein `doc`-Attribut. Optionale Felder sind als Union `["null", <typ>]` mit `"default": null` modelliert.
- Für Geldbeträge `decimal` als `bytes` mit `precision` und `scale` verwenden, nie `float` oder `double`.
- Für Zeitpunkte `timestamp-millis` (UTC), für Kalenderdaten `date`.
- Enums nur für stabile, abschliessende Wertemengen verwenden und immer ein `default`-Symbol (z. B. `UNKNOWN`) definieren.
- Keine Personendaten ins Event, die der Consumer nicht zwingend braucht (Datensparsamkeit gemäss DSG).

### 4. Versionierung und Kompatibilität

- Kompatibilitätsmodus in der Schema Registry: **`BACKWARD_TRANSITIVE`** als Standard für das Subject.
- Kompatible Änderungen (Minor, gleiches Topic):
  - optionales Feld mit Default hinzufügen
  - Feld mit Default entfernen
  - `doc` ändern, Alias hinzufügen
- Inkompatible Änderungen (Major, neues Topic `….v2` und neuer Namespace `….v2`):
  - Pflichtfeld ohne Default hinzufügen
  - Feld umbenennen ohne Alias, Typ ändern
  - Semantik eines Feldes ändern
- Bei einer Major-Version produziert der Producer während einer vereinbarten Übergangsfrist auf beide Topics. Die Abkündigung der alten Version wird den Consumern aktiv mitgeteilt.
- Kompatibilität vor dem Merge automatisiert prüfen (z. B. Schema-Registry-Maven-Plugin mit `test-compatibility`).

### 5. Dokumentieren und reviewen

- Topic, Schema, Owner und Consumer im Event-Katalog bzw. in der AsyncAPI-Spezifikation nachführen.
- Neues Topic oder Major-Version: als ADR in der arc42-Doku des Producers festhalten.
- Schema-Änderungen nur per Pull Request mit Review durch mindestens eine Person aus einem Consumer-Team.

## Qualitätskriterien

- **Name:** Event-Name ist ein fachliches Ereignis in der Vergangenheitsform, Topic-Name folgt der Konvention.
- **Ownership:** Genau ein Producer-Team ist als Owner dokumentiert.
- **Selbsterklärend:** Jedes Feld hat einen sprechenden Namen und ein `doc`; das Event ist ohne Rückfrage verständlich.
- **Kompatibilität:** Die Änderung ist gemäss `BACKWARD_TRANSITIVE` kompatibel oder als neue Major-Version mit eigenem Topic umgesetzt.
- **Ordnung:** Der Message-Key garantiert die Reihenfolge pro Aggregat.
- **Typsicherheit:** Korrekte Logical Types für Beträge, Datum und Zeit; keine Fliesskommazahlen für Geld.
- **Datenschutz:** Nur notwendige Personendaten, sensitive Daten nur nach Abklärung mit Security und Datenschutz.
- **Idempotenz:** Consumer können anhand von `eventId` Duplikate erkennen.
