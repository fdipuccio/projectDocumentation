MODULE: PM VERSION: 1
## 1. Obiettivo
Sviluppare un backend per un sistema di autenticazione utenti che permetta la registrazione, il login tramite email e password, la generazione di token JWT per sessioni sicure, la protezione di endpoint tramite autenticazione e la gestione base del reset della password.

## 2. Contesto e vincoli
- Il backend deve essere implementato utilizzando il framework FastAPI (Python).
- Il database da utilizzare è PostgreSQL.
- Il sistema deve supportare la generazione di token JWT per l'autenticazione.
- Devono essere esposti endpoint REST protetti mediante autenticazione tramite JWT.
- Stack fornito nell’ambiente attuale è Java, Rest API, PostgreSQL, JWT: questa tecnologia non è conforme al vincolo “Deve usare FastAPI”, pertanto si tratta di un ambito di divergenza da chiarire o di un contesto esistente da integrare, non di uno stack da usare direttamente.
  
## 3. Assunzioni
- La registrazione richiede email e password come dati minimi.
- Il reset password verrà implementato in modalità base, senza integrazione con sistemi di invio email o autenticazione multi-fattore, a meno di ulteriori specifiche.
- Il sistema non include funzionalità di gestione ruoli o permessi avanzati ma solo protezione tramite autenticazione.
- L’ambiente di deploy e le infrastrutture di rete e sicurezza (SSL, CORS, ecc.) sono gestite esternamente e non fanno parte del perimetro MVP.
- Non è richiesta una UI, solo backend API REST.
- Si assume che la generazione e verifica del JWT seguirà standard comuni (es. RS256 o HS256) senza personalizzazioni particolari.

## 4. Scope MVP
- Endpoint per registrazione utente con validazione base dei dati.
- Endpoint per login con email e password, che ritorna token JWT.
- Implementazione gestione JWT: generazione, firma, verifica.
- Endpoint protetti autenticati tramite verifica token JWT.
- Endpoint per reset password base (es. reset via token temporaneo generato e validato).
- Persistenza dati utenti e dati di autenticazione su database PostgreSQL.
- Documentazione API base (es. OpenAPI/Swagger) generata da FastAPI.

## 5. Out of scope
- Sviluppo frontend o interfacce utente.
- Integrazione con servizi di invio email o SMS per reset password.
- Funzionalità OAuth o autenticazione social.
- Gestione avanzata di ruoli, permessi e autorizzazioni.
- Logging centralizzato, monitoring e alerting.
- Gestione di sicurezza avanzata come rate limiting, protezioni specifiche contro attacchi.
- Automazione di deploy e configurazioni infrastrutturali.

## 6. Task tecnici ordinati
1. Setup ambiente di sviluppo FastAPI e connessione con database PostgreSQL.
2. Definizione modello dati utenti e tabelle PostgreSQL.
3. Implementazione endpoint registrazione utente con validazione dati e persistenza.
4. Implementazione endpoint login con verifica email/password.
5. Implementazione generazione e firma token JWT in fase di login.
6. Implementazione middleware o dependency per protezione endpoint tramite verifica JWT.
7. Creazione endpoint protetto di esempio per validare meccanismo autenticazione.
8. Implementazione base endpoint reset password con generazione token temporanei.
9. Testing unitario e di integrazione su tutte le funzionalità implementate.
10. Documentazione API tramite FastAPI OpenAPI schema.
11. Revisione sicurezza base (hash password, gestione segreti JWT).

## 7. Acceptance criteria
- L’utente può registrarsi fornendo email e password valide e viene salvato correttamente nel database.
- L’utente può effettuare login con credenziali corrette e ricevere un token JWT valido.
- Gli endpoint protetti non sono accessibili senza token JWT valido.
- È possibile effettuare il reset password tramite endpoint dedicato, con generazione e validazione token reset.
- Tutte le funzionalità sono implementate usando FastAPI e PostgreSQL come database.
- È garantita la persistenza dei dati utenti nel database PostgreSQL.
- La documentazione API è accessibile e descrive correttamente ogni endpoint.
- I test di unità e integrazione sono presenti e superati con successo.

## 8. Rischi e punti aperti
- Ambiguità nello stack: requisito dichiara obbligo uso FastAPI e PostgreSQL, ma stack fornito contiene Java e REST API. Necessaria chiarificazione su come integrare o sostituire stack esistente.
- Dettaglio limitato su reset password: modalità di comunicazione all’utente (es. email) non definita, va approfondito o specificato per implementazione completa.
- Non sono indicati requisiti specifici su algoritmo di hashing password e algoritmo di firma JWT.
- Mancanza di specifica su politiche di sicurezza (es. durata token, criteri passwd, lockout) da definire.
- Non è menzionata la gestione di utenti duplicati o casi speciali (es. utenti sospesi).
- Ambiente di deploy e configurazione operativa non inclusi, da coordinare con team infrastrutture.