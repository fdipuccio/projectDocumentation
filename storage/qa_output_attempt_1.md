MODULE: QA
VERSION: 1
FINAL_STATUS: REJECTED

## 1. Checklist — Copertura requisiti
* [SI] Gli endpoint REST devono ricevere ordini con tutte le informazioni necessarie per l'elaborazione — La proposta Backend copre l'elaborazione degli ordini tramite eventi.
* [SI] Stripe deve essere correttamente integrato e gestire correttamente i pagamenti, incluse situazioni di retry — La proposta specifica l'integrazione con Stripe e l'utilizzo di chiavi di idempotenza.
* [SI] Il workflow di elaborazione ordine deve seguire la sequenza definita con meccanismi di compensazione operativi — Meccanismi di idempotenza e retry sono presenti.
* [SI] RabbitMQ deve gestire efficacemente gli eventi asincroni di aggiornamento stato ordine — È menzionato l'uso di RabbitMQ per la gestione degli eventi ordine e pagamento.
* [PARZIALE] L'integrazione logistica deve fornire correttamente il tracking delle spedizioni — Integrato nei moduli, ma senza dettagli su come il tracking viene effettuato.
* [NO]* Notifiche devono essere inviate a ogni cambio di stato ordine — Manca una descrizione dettagliata sull'implementazione delle notifiche per ogni cambio di stato.
* [SI] Le API di backoffice devono permettere la gestione degli ordini e dei rimborsi — La gestione ordini è parte del processo automatizzato.
* [SI] Il sistema di gestione magazzino deve scalare e ripristinare stock correttamente — Non menzionato esplicitamente, ma implicato nel flusso di elaborazione ordine.

## 2. Checklist — Contratti API
* [NO] Ogni endpoint ha metodo HTTP, path, request schema, response schema e esempio concreto — Non sono disponibili dettagli contratti API nell'implementazione proposta.
* [NO] Le regole di validazione sono esplicite per ogni campo (tipo, formato, obbligatorietà, regex ove necessario) — Mancante nella proposta.
* [NO] Il formato degli errori è consistente tra tutti gli endpoint (struttura JSON uniforme) — Mancante nella documentazione proposta.
* [NO] I codici HTTP di risposta (2xx, 4xx, 5xx) sono specificati per ogni endpoint — Non definiti nella proposta.
* [NO] La strategia di paginazione è definita per le liste (se applicabile) — Non applicabile poiché mancano dettagli su liste e paginazione.

## 3. Checklist — Business logic e scenari limite
* [PARZIALE] I flussi principali sono descritti passo per passo (non solo a parole generiche) — Flussi di consumo eventi sono descritti, ma mancano dettagli approfonditi.
* [PARZIALE] Gli scenari di concorrenza sono trattati (es. doppia registrazione, doppio click, race condition) — La gestione di concorrenza è citata come rischio ma non dettagliatamente affrontata.
* [SI] I casi di fallimento delle integrazioni esterne hanno una strategia (retry, fallback, circuit breaker) — Sono presenti strategie di retry con backoff esponenziale.
* [NO] Le regole di business critiche sono esplicite e non ambigue — Mancano specifiche dettagliate delle regole di business nei documenti.

## 4. Checklist — Persistenza e schema dati
* [NO] Le tabelle/collezioni principali sono definite con i campi e i tipi — Non sono dettagliate nella proposta.
* [NO] Gli indici sono specificati per le colonne usate in query frequenti o join — Non menzionati nella proposta.
* [NO] I vincoli di unicità e foreign key sono dichiarati — Non specificati.
* [NO] La strategia di migrazione dello schema è menzionata — Non citata.

## 5. Checklist — Strategia di test
* [SI] Esistono test per i happy path di ogni funzionalità principale — I test unitari coprono i consumatori di eventi.
* [PARZIALE] Esistono test per i casi di errore critici (validazione fallita, not found, unauthorized) — Presenti ma non dettagliati.
* [SI] Esistono test di integrazione per le dipendenze esterne (DB, servizi esterni) — Sono presenti test di integrazione con Stripe e RabbitMQ.
* [PARZIALE] I test specificano input e expected output concreti (non generici) — Alcuni dettagli forniti, ma non per tutti i casi.

## 6. Requisiti mancanti
Notifiche devono essere inviate a ogni cambio di stato ordine.

## 7. Rischi e problemi
1. Ritardi nelle Integrazioni Esterne: ALTA — Latenze con Stripe e servizi logistici possono impattare il flusso ordine.
2. Gestione Concorrenza: MEDIA — Mancanza di dettagli su come gestire situazioni concorrenti.

## 8. Azioni richieste
[PRIORITÀ ALTA] Dettagliare l'implementazione delle notifiche per ogni cambio di stato ordine.
[PRIORITÀ ALTA] Fornire dettagli sui contratti API con esplicitazione di metodi HTTP, path, schema richieste e risposte.
[PRIORITÀ ALTA] Descrivere le tabelle di persistazione e le relazioni tra esse, inclusi indici e vincoli.
[PRIORITÀ MEDIA] Definire le strategie di gestione della concorrenza e scenari con alta simultaneità.
[PRIORITÀ MEDIA] Fornire dettagli sulle regole di business critiche ed esplicitare i flussi passo per passo.
