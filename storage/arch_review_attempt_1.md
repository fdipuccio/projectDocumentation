MODULE: ARCH_REVIEW
VERSION: 1
ARCH_STATUS: REJECTED

## 1. Checklist — Completezza architetturale
* `SI` Pattern architetturale scelto e giustificato
* `NO` Component diagram con componenti e responsabilità
* `NO` Contratti tra moduli con formati dati
* `SI` Decisioni architetturali documentate (ADR)
* `SI` Linee guida azionabili per gli specialisti

## 2. Checklist — Coerenza con i requisiti PM
* `NO` Task tecnici del PM indirizzabili dall'architettura
* `SI` Acceptance criteria del PM supportati
`SI` Assunzioni non contraddicono vincoli PM

## 3. Checklist — Qualità del design
* `NO` Confini tra moduli chiari per gli specialisti
* `NO` Contratti con dettaglio sufficiente (campi, tipi, formati)
`SI` Pattern appropriato per worker
`SI` Aspetti cross-cutting indirizzati (sicurezza, logging, errori)

## 4. Checklist — Qualità architetturale
Ogni gate è critico (*).
* `SI` IMPLEMENTABILITÀ: realizzabile con lo stack dichiarato
* `SI` SCALABILITÀ: supporta crescita o vincoli esplicitati
* `SI` TESTABILITÀ: layer testabili in isolamento
* `SI` MODULARITÀ: responsabilità singola per componente
* `SI` DISACCOPPIAMENTO: dipendenze unidirezionali, no circular
* `SI` SICUREZZA: authn/authz, secret management, input validation

## 5. Azioni richieste
[PRIORITÀ ALTA] Creare e documentare un diagramma dei componenti con responsabilità chiare.
[PRIORITÀ ALTA] Definire e documentare i contratti tra moduli con dettagli su campi, tipi, e formati.
[PRIORITÀ ALTA] Assicurare che tutti i task tecnici del PM siano coperti dall'architettura proposta.