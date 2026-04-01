MODULE: PM VERSION: 1
## 1. Obiettivo
Sviluppare un backend per un sistema di autenticazione utenti che consenta la registrazione, il login tramite email e password, la generazione di token JWT per autenticazione protetta, la gestione di endpoint protetti e una funzionalità base di reset password.

## 2. Contesto e vincoli
- Il backend deve essere implementato utilizzando il framework FastAPI.
- Il database per la persistenza dei dati utenti deve essere PostgreSQL.
- Il sistema dovrà gestire autenticazione mediante token JWT.
- Il reset password previsto è di complessità "base" (non è ulteriormente specificato il flusso o metodo).
- Stack tecnologico definito: Python, FastAPI, PostgreSQL, JWT.

## 3. Assunzioni
- Per "reset password base" si assume un flusso semplificato, ad esempio invio di token o link via email, ma senza specifiche di sicurezza aggiuntive o meccanismi avanzati.
- La gestione degli utenti (es. ruoli, autorizzazioni) non è richiesta oltre l'autenticazione.
- Non è previsto il supporto per autenticazione multi-fattore o social login.
- Non sono richieste specifiche su policy password o validazioni avanzate.
- Endpoint protetti saranno accessibili solo tramite token JWT validi.
- Il sistema di invio email per reset password (se previsto) è disponibile o verrà integrato successivamente (non esplicitato tra i vincoli).
- Non ci sono indicazioni su scadenza token o norme di sicurezza aggiuntive.
- Il sistema deve essere predisposto per ambienti enterprise ma non sono specificate esigenze di scalabilità o alta disponibilità.

## 4. Scope MVP
- Endpoint per registrazione utente con validazione base dati.
- Endpoint per login utente con email e password.
- Generazione e restituzione di token JWT al login.
- Middleware o dipendenza per proteggere endpoint specifici con token JWT.
- Endpoint per reset password base (es. richiesta reset e validazione token/reset tramite link).
- Persistenza dati utenti su PostgreSQL.
- Documentazione basica dell’API (ad esempio con OpenAPI/Swagger generata da FastAPI).

## 5. Out of scope
- Implementazione di MFA (Multi-Factor Authentication).
- Gestione ruoli o autorizzazioni avanzate.
- Funzionalità di registrazione social o altri metodi di login.
- Gestione dettagliata della sicurezza avanzata (es. limit login, blocco account, log attività).
- Scenari di alta disponibilità, load balancing o infrastruttura di deployment.
- Sistemi di email server o mailing list (andrà integrato da altro team o successivamente).
- UI/frontend per interagire col sistema.

## 6. Task tecnici ordinati
1. Progettazione schema database utenti su PostgreSQL (email, password hash, token reset, timestamp, ecc.).
2. Configurazione ambiente FastAPI con connessione a PostgreSQL.
3. Implementazione endpoint registrazione utenti con validazioni base.
4. Implementazione endpoint login email/password con verifica credenziali.
5. Integrazione libreria JWT per generazione token al login.
6. Implementazione meccanismo di protezione endpoint con token JWT (middleware / dipendenza).
7. Implementazione endpoint reset password base (richiesta reset e validazione token).
8. Scrittura test automatici per validazione endpoint e sicurezza base.
9. Documentazione API e relative specifiche swagger.
10. Revisione sicurezza base (es. hashing password sicuro, gestione segreti).

## 7. Acceptance criteria
- Registrazione utente crea nuovo record in PostgreSQL con password salvata in modo sicuro.
- Login con email e password corretti restituisce token JWT valido.
- Endpoint protetti rispondono con errore 401 se token assente o non valido.
- Endpoint reset password permette almeno il flusso base di richiesta e validazione reset.
- Tutte le funzioni sono raggiungibili tramite API FastAPI documentate.
- Test automatici di unità/integrativi coprono almeno i flussi di registrazione, login e reset password.
- Implementazione rispetta requisiti tecnici e stack definiti (FastAPI, PostgreSQL, JWT).
- Codice base è strutturato in modo riutilizzabile per futuri ampliamenti.

## 8. Rischi e punti aperti
- Ambiguità sul dettaglio del flusso reset password ("base" non definito nello specifico).
- Non definito sistema o infrastruttura per invio email di reset password.
- Nessuna indicazione sulla gestione della sicurezza avanzata o politiche password.
- Potenziali richieste future di funzionalità non previste (MFA, ruoli, social login).
- Mancanza di specifiche su configurazione token JWT (scadenza, algoritmo).
- Non è definito se il sistema deve gestire più ambienti o integrazione con sistemi esterni.
- Necessità di chiarimenti su volumi attesi e requisiti prestazionali per dimensionare backend.