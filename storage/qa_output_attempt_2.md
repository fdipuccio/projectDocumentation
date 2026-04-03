MODULE: QA
VERSION: 1
FINAL_STATUS: REJECTED

## 1. Checklist — Copertura requisiti
[NO] Utente può registrarsi con email unica e password conforme a policy — Non è possibile verificare la concreta implementazione nella proposta backend a causa della mancanza del documento.
[NO] Utente può effettuare login con email/password valide e ricevere token JWT con scadenza — Mancanza del dettaglio implementativo confermato.
[NO] Endpoint protetti rispondono solo se token JWT è presente, valido e non scaduto; altrimenti ritornano errore 401 — Informazioni incomplete per validazione.
[NO] Reset password consente di inviare codice/link via email e permette modifica password — Non verificabile.
[NO] Password salvate nel database sono hashed con salt sicuro — Mancata conferma.
[NO] Tutte le API validano gli input e impediscono injection o dati non validi — Mancanza di dettagli.
[NO] Log eventi critici sono correttamente registrati e consultabili in modo sicuro — Mancanza di conferma.
[NO] Il sistema risponde con performance accettabili — Non valutabile.
[NO] Il sistema scala orizzontalmente senza perdita di funzionalità — Mancata prova documentata.
[NO] Test di sicurezza base superati — Insufficienti evidenze.
[NO] Documentazione tecnica completa e aggiornata — Assenza documenti.

## 2. Checklist — Contratti API
[NO] Ogni endpoint ha metodo HTTP, path, request schema, response schema e esempio concreto — Mancante.
[NO] Le regole di validazione sono esplicite per ogni campo — Mancante.
[NO] Il formato degli errori è consistente tra tutti gli endpoint — Mancante.
[NO] I codici HTTP di risposta sono specificati per ogni endpoint — Mancante.
[NO] La strategia di paginazione è definita per le liste (se applicabile) — Non specificato.

## 3. Checklist — Business logic e scenari limite
[NO] I flussi principali sono descritti passo per passo — Mancanti.
[NO] Gli scenari di concorrenza sono trattati — Non presenti.
[NO] I casi di fallimento delle integrazioni esterne hanno una strategia — Mancata definizione.
[NO] Le regole di business critiche sono esplicite e non ambigue — Non presenti.

## 4. Checklist — Persistenza e schema dati
[NO] Le tabelle/collezioni principali sono definite con i campi e i tipi — Non esplicitate.
[NO] Gli indici sono specificati per le colonne usate in query frequenti o join — Mancante.
[NO] I vincoli di unicità e foreign key sono dichiarati — Mancanti.
[NO] La strategia di migrazione dello schema è menzionata — Non presente.

## 5. Checklist — Strategia di test
[NO] Esistono test per i happy path di ogni funzionalità principale — Non documentato.
[NO] Esistono test per i casi di errore critici — Non documentato.
[NO] Esistono test di integrazione per le dipendenze esterne — Mancante.
[NO] I test specificano input e expected output concreti — Non specificato.

## 6. Requisiti mancanti
Tutti i requisiti specificati nella specifica PM non sono stati trovati o confermati nella proposta backend fornita.

## 7. Rischi e problemi
Non è stato possibile valutare i rischi tecnici per mancanza di documentazione.

## 8. Azioni richieste
[PRIORITÀ ALTA] Fornire la documentazione completa della proposta backend conforme alla specifica PM inclusiva di dettagli su API, schemi dati, business logic e test.
[PRIORITÀ ALTA] Includere esempi concreti e codici di risposta HTTP per tutti gli endpoint.
[PRIORITÀ ALTA] Dettagliare le strategie di sicurezza e gestione degli errori.
[PRIORITÀ MEDIA] Documentare la gestione della concorrenza e strategie di fallback.
[PRIORITÀ MEDIA] Fornire piano di test dettagliato con input/output attesi.
[PRIORITÀ BASSA] Specificare strategie di migrazione e indici di database.

La mancanza totale o parziale di informazioni chiave impedisce la valutazione positiva e determina il rifiuto della proposta.