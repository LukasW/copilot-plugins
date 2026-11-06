# Namenskonventionen für Kafka-Events

Referenz zum Skill `kafka-event-design`. Diese Konventionen gelten für alle neuen Topics und Schemas. Bestehende Topics werden erst bei der nächsten Major-Version angepasst.

## Topics

**Muster:** `<domain>.<subdomain>.<entity>.<event-type>.v<major>`

| Teil         | Regel                                                        | Beispiel        |
|--------------|--------------------------------------------------------------|-----------------|
| `domain`     | Fachliche Domäne, lowercase                                  | `policy`        |
| `subdomain`  | Bounded Context innerhalb der Domäne, lowercase              | `contract`      |
| `entity`     | Aggregat im Singular, lowercase, Bindestriche erlaubt        | `policy`        |
| `event-type` | `events` (Domain-Events), `state` (compacted Zustand), `commands` | `events`    |
| `v<major>`   | Major-Version des Schemas                                    | `v1`            |

**Beispiele:**

- `policy.contract.policy.events.v1` — alle Domain-Events zu Policen
- `claims.benefit.claim.state.v1` — compacted Topic mit aktuellem Zustand je Leistungsfall
- `partner.customer.address.events.v2` — Adress-Events in Major-Version 2

**Weitere Regeln:**

- Nur `a-z`, `0-9`, `.` und `-`. Keine Underscores (kollidieren in Kafka-Metriken mit Punkten).
- Keine Umgebungsnamen im Topic (`dev`, `prod`); die Umgebung ergibt sich aus dem Cluster.
- Keine Team- oder Applikationsnamen im Topic; Teams und Applikationen ändern sich, die Fachlichkeit bleibt.
- Retry- und Dead-Letter-Topics hängen ein Suffix an: `<topic>.retry`, `<topic>.dlt`, und gehören dem Consumer.

## Events (Avro-Record-Namen)

- **PascalCase**, Muster `<Entity><VerbImPartizip>`: `PolicyCreated`, `PolicyCancelled`, `ClaimSettled`, `AddressChanged`.
- Vergangenheitsform, fachliche Sprache der Domäne (Englisch).
- Keine technischen Verben wie `Updated`, `Inserted`, `Saved`, wenn es ein fachlich präziseres Verb gibt (`PremiumAdjusted` statt `PolicyUpdated`).
- Keine Präfixe oder Suffixe wie `Event`, `Msg`, `Dto`.

## Avro-Namespaces

**Muster:** `ch.css.<domain>.events.v<major>`

- Beispiel: `ch.css.policy.events.v1`
- Gemeinsame Typen (z. B. `EventMetadata`, `MonetaryAmount`) liegen in `ch.css.common.events`.
- Die Major-Version im Namespace entspricht der Major-Version im Topic.

## Schema-Registry-Subjects

- Subject-Naming-Strategy: `TopicRecordNameStrategy`, damit mehrere Event-Typen auf einem Topic möglich sind.
- Daraus ergibt sich das Subject `<topic>-<fully.qualified.RecordName>`, z. B. `policy.contract.policy.events.v1-ch.css.policy.events.v1.PolicyCreated`.

## Felder

- **camelCase**: `policyId`, `validFrom`, `grossPremium`.
- IDs enden auf `Id`: `policyId`, `customerId`. Keine Abkürzungen wie `polNr`.
- Booleans als Aussage mit `is`/`has`: `isActive`, `hasDeductible`.
- Zeitpunkte enden auf `Time` (`timestamp-millis`, UTC): `eventTime`, `cancellationTime`.
- Kalenderdaten enden auf `Date` oder beschreiben einen Gültigkeitsbeginn bzw. ein Gültigkeitsende (`date`): `birthDate`, `validFrom`, `validTo`.
- Beträge als Record `MonetaryAmount` mit `amount` (`decimal`, `precision` 18, `scale` 2) und `currency` (ISO 4217).
- Listen im Plural: `coverages`, `insuredPersons`.

## Enums

- Enum-Name in PascalCase (`PolicyStatus`), Symbole in `UPPER_SNAKE_CASE` (`ACTIVE`, `SUSPENDED`).
- Immer ein `default`-Symbol (in der Regel `UNKNOWN`) definieren, damit neue Symbole für bestehende Consumer kompatibel bleiben.

## Message-Headers

| Header           | Inhalt                                           |
|------------------|--------------------------------------------------|
| `ce_id`          | Eindeutige Event-ID (UUID), identisch mit `metadata.eventId` |
| `ce_type`        | Voll qualifizierter Record-Name                  |
| `ce_source`      | Produzierendes System                            |
| `traceparent`    | W3C-Trace-Context für Distributed Tracing        |

Die Header orientieren sich am CloudEvents-Kafka-Binding, damit Events auch ausserhalb von Kafka einheitlich beschrieben werden können.
