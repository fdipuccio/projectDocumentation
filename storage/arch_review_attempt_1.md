```markdown
MODULE: ARCH_REVIEW
VERSION: 1
ARCH_STATUS: APPROVED

## 1. Checklist — Completezza architetturale
* `[SI]` Pattern architetturale scelto e giustificato
* `[SI]` Component diagram con componenti e responsabilità
* `[NO]` Contratti tra moduli con formati dati
* `[SI]` Decisioni architetturali documentate (ADR)
* `[SI]` Linee guida azionabili per gli specialisti

## 2. Checklist — Coerenza con i requisiti PM
* `[SI]` Task tecnici del PM indirizzabili dall'architettura
* `[SI]` Acceptance criteria del PM supportati
`[SI]` Assunzioni non contraddicono vincoli PM

## 3. Checklist — Qualità del design
* `[SI]` Confini tra moduli chiari per gli specialisti
* `[NO]` Contratti con dettaglio sufficiente (campi, tipi, formati)
`[SI]` Pattern appropriato per worker
`[SI]` Aspetti cross-cutting indirizzati (sicurezza, logging, errori)

## 4. Checklist — Qualità architetturale
* `[SI]` IMPLEMENTABILITÀ: realizzabile con lo stack dichiarato
* `[SI]` SCALABILITÀ: supporta crescita o vincoli esplicitati
* `[SI]` TESTABILITÀ: layer testabili in isolamento
* `[SI]` MODULARITÀ: responsabilità singola per componente
* `[SI]` DISACCOPPIAMENTO: dipendenze unidirezionali, no circular
* `[SI]` SICUREZZA: authn/authz, secret management, input validation

## 5. Azioni richieste
Se REJECTED: `[PRIORITÀ ALTA] Definire completamente i contratti e fornire dettagli sui formati dati tra moduli.` per ogni problema.
Se APPROVED: `Nessuna`.
```