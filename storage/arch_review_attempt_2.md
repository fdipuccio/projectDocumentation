MODULE: ARCH_REVIEW
VERSION: 1
ARCH_STATUS: REJECTED

## 1. Checklist — Completezza architetturale
* `[SI]` Pattern architetturale scelto e giustificato
* `[SI]` Component diagram con componenti e responsabilità
* `[NO]` Contratti tra moduli con formati dati - Dettagli insufficienti sugli schemi degli eventi nel sistema RabbitMQ.
* `[SI]` Decisioni architetturali documentate (ADR)
* `[SI]` Linee guida azionabili per gli specialisti

## 2. Checklist — Coerenza con i requisiti PM
* `[SI]` Task tecnici del PM indirizzabili dall'architettura
* `[SI]` Acceptance criteria del PM supportati
* `[SI]` Assunzioni non contraddicono vincoli PM

## 3. Checklist — Qualità del design
* `[SI]` Confini tra moduli chiari per gli specialisti
* `[NO]` Contratti con dettaglio sufficiente (campi, tipi, formati) - Definizioni incomplete nei contratti per API e eventi.
* `[SI]` Pattern appropriato per worker
* `[SI]` Aspetti cross-cutting indirizzati (sicurezza, logging, errori)

## 4. Checklist — Qualità architetturale
Ogni gate è critico (*).
* `[SI]` IMPLEMENTABILITÀ: realizzabile con lo stack dichiarato
* `[SI]` SCALABILITÀ: supporta crescita o vincoli esplicitati
* `[SI]` TESTABILITÀ: layer testabili in isolamento
* `[NO]` MODULARITÀ: responsabilità singola per componente - Alcuni componenti sembrano avere sovrapposizioni nelle responsabilità.
* `[SI]` DISACCOPPIAMENTO: dipendenze unidirezionali, no circular
* `[SI]` SICUREZZA: authn/authz, secret management, input validation

## 5. Azioni richieste
[PRIORITÀ ALTA] Specificare i dettagli degli schemi degli eventi nel sistema RabbitMQ.
[PRIORITÀ ALTA] Definire chiaramente i contratti API con campi e tipi completi.
[PRIORITÀ MEDIA] Verificare e correggere eventuali sovrapposizioni di responsabilità tra i componenti dell'architettura.