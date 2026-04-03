MODULE: QA
VERSION: 1
FINAL_STATUS: REJECTED

## 1. Checklist — Copertura requisiti
[NO] Utente può registrarsi con email unica e password conforme a policy — Non è possibile verificare la copertura perché non è disponibile il contenuto della proposta backend.
[NO] Utente può effettuare login con email/password valide e ricevere token JWT con scadenza — Non è possibile verificare la copertura perché non è disponibile il contenuto della proposta backend.
[NO] Endpoint protetti rispondono solo se token JWT è presente, valido e non scaduto; altrimenti ritornano errore 401 — Non è possibile verificare la copertura perché non è disponibile il contenuto della proposta backend.
[NO] Reset password consente di inviare codice/link via email e permette modifica password — Non è possibile verificare la copertura perché non è disponibile il contenuto della proposta backend.
[NO] Password salvate nel database sono hashed con salt sicuro — Non è possibile verificare la copertura perché non è disponibile il contenuto della proposta backend.
[NO] Tutte le API validano gli input e impediscono injection o dati non validi — Non è possibile verificare la copertura perché non è disponibile il contenuto della proposta backend.
[NO] Log eventi critici sono correttamente registrati e consultabili in modo sicuro — Non è possibile verificare la copertura perché non è disponibile il contenuto della proposta backend.
[NO] Il sistema risponde con performance accettabili (es. tempi di risposta inferiori a 1 secondo per endpoint critici) — Non è possibile verificare la copertura perché non è disponibile il contenuto della proposta backend.
[NO] Il sistema scala orizzontalmente senza perdita di funzionalità — Non è possibile verificare la copertura perché non è disponibile il contenuto della proposta backend.
[NO] Test di sicurezza base superati (es. tentativi ripetuti login bloccati o limitati) — Non è possibile verificare la copertura perché non è disponibile il contenuto della proposta backend.
[NO] Documentazione tecnica completa e aggiornata — Non è possibile verificare la copertura perché non è disponibile il contenuto della proposta backend.

## 2. Checklist — Contratti API
[NO] Ogni endpoint ha metodo HTTP, path, request schema, response schema e esempio concreto — Nessuna evidenza disponibile.
[NO] Le regole di validazione sono esplicite per ogni campo (tipo, formato, obbligatorietà, regex ove necessario) — Nessuna evidenza disponibile.
[NO] Il formato degli errori è consistente tra tutti gli endpoint (struttura JSON uniforme) — Nessuna evidenza disponibile.
[NO] I codici HTTP di risposta (2xx, 4xx, 5xx) sono specificati per ogni endpoint — Nessuna evidenza disponibile.
[NO] La strategia di paginazione è definita per le liste (se applicabile) — Nessuna evidenza disponibile.

## 3. Checklist — Business logic e scenari limite
[NO] I flussi principali sono descritti passo per passo (non solo a parole generiche) — Nessuna evidenza disponibile.
[NO] Gli scenari di concorrenza sono trattati (es. doppia registrazione, doppio click, race condition) — Nessuna evidenza disponibile.
[NO] I casi di fallimento delle integrazioni esterne hanno una strategia (retry, fallback, circuit breaker) — Nessuna evidenza disponibile.
[NO] Le regole di business critiche sono esplicite e non ambigue — Nessuna evidenza disponibile.

## 4. Checklist — Persistenza e schema dati
[NO] Le tabelle/collezioni principali sono definite con i campi e i tipi — Nessuna evidenza disponibile.
[NO] Gli indici sono specificati per le colonne usate in query frequenti o join — Nessuna evidenza disponibile.
[NO] I vincoli di unicità e foreign key sono dichiarati — Nessuna evidenza disponibile.
[NO] La strategia di migrazione dello schema è menzionata — Nessuna evidenza disponibile.

## 5. Checklist — Strategia di test
[NO] Esistono test per i happy path di ogni funzionalità principale — Nessuna evidenza disponibile.
[NO] Esistono test per i casi di errore critici (validazione fallita, not found, unauthorized) — Nessuna evidenza disponibile.
[NO] Esistono test di integrazione per le dipendenze esterne (DB, servizi esterni) — Nessuna evidenza disponibile.
[NO] I test specificano input e expected output concreti (non generici) — Nessuna evidenza disponibile.

## 6. Requisiti mancanti
Tutta la proposta backend non è stata fornita e quindi non è possibile verificare i requisiti.

## 7. Rischi e problemi
- ALTA: Mancanza di informazioni – impossibile eseguire QA.
- ALTA: Rischio di non conformità alla specifica PM se la proposta backend mancante.

## 8. Azioni richieste
[ALTA] Fornire il documento completo della proposta backend per poter eseguire la revisione di qualità puntuale secondo la specifica PM.