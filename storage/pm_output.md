MODULE: PM VERSION: 1
## 1. Obiettivo
Sviluppare un backend per un sistema di autenticazione utenti che consenta la registrazione, login tramite email e password, generazione e gestione di token JWT per sessioni protette e una funzionalità base di reset della password.

## 2. Contesto e vincoli
- Il backend deve essere implementato utilizzando il framework FastAPI.
- Il database deve essere PostgreSQL per la gestione della persistenza dati.
- Il sistema utilizzerà JWT (JSON Web Token) per la generazione e verifica dei token di autenticazione.
- Il progetto si concentra esclusivamente sulle funzionalità di autenticazione/designed per essere integrato in sistemi più ampi.
- Il reset password previsto è una funzionalità base: non è specificato se deve prevedere email di reset o altre modalità complesse.

## 3. Assunzioni
- Il sistema di autenticazione sarà usato in un contesto enterprise, quindi si assume un livello base di sicurezza adeguato al contesto (ad esempio hashing sicuro delle password).
- Non ci sono requisiti sul sistema di gestione utenti (es. ruoli o permessi), quindi tali funzionalità non sono incluse.
- Le email degli utenti sono univoche e vengono utilizzate come chiave per l’identificazione.
- Non è esplicitamente richiesto il supporto per autenticazioni social o multifattore.
- La gestione dei token JWT deve includere almeno creazione e verifica, ma non sono stati specificati dettagli sul tempo di scadenza o rotazione token.

## 4. Scope MVP
- Endpoint per la registrazione utente con email e password.
- Endpoint login che accetta email e password, restituisce token JWT valido.
- Implementazione della generazione di token JWT con configurazione minima (es. segreto e algoritmo).
- Endpoint protetti che richiedono token JWT per l’accesso.
- Funzionalità base di reset password: interface e meccanismo che consenta agli utenti di modificare la password (dettagli da definire, probabilmente tramite token o codice temporaneo).
- Persistenza dati utenti su PostgreSQL con struttura adeguata a gestire email, password hashed e eventuali token di reset.

## 5. Out of scope
- Implementazione di sistemi di verifica email o email di conferma.
- Supporto per autenticazione tramite OAuth o social login.
- Gestione ruoli, permessi o livelli di accesso diversi tra utenti.
- Funzionalità avanzate di reset password, come invio automatico di email con link/reset token.
- Scalabilità orizzontale o deployment.
- Monitoraggio, logging esteso o auditing delle autentificazioni.
- UI o client frontend.

## 6. Task tecnici ordinati
1. Definizione schema database utenti in PostgreSQL (campi: id, email unico, password hashed, eventuali token/reset token).
2. Implementazione modello dati e ORM (es. SQLAlchemy o equivalente).
3. Setup dell’ambiente FastAPI con configurazione base.
4. Implementazione endpoint di registrazione utente con validazioni base (es. email valida, password minima).
5. Implementazione hashing sicuro delle password (es. bcrypt o argon2).
6. Implementazione endpoint login che verifica credenziali e genera token JWT.
7. Configurazione e generazione token JWT con algoritmo e chiave segreta.
8. Creazione middleware/dependency per protezione endpoint con verifica JWT.
9. Implementazione endpoint protetti di esempio per validazione accessi.
10. Implementazione endpoint/reset password base (es. invio token/reset password).
11. Gestione sicurezza per reset password (validità token, modifica password).
12. Testing di integrazione per tutti gli endpoint.
13. Documentazione API con OpenAPI/Swagger.

## 7. Acceptance criteria
- Il backend permette la registrazione di un utente con email unica e password hashed.
- L’endpoint login accetta email e password corretti e restituisce un token JWT valido.
- I token JWT seguono lo standard e sono verificabili.
- Gli endpoint protetti rifiutano richieste senza token o con token non valido.
- L’utente può richiedere reset della password e modificarla utilizzando la procedura base prevista.
- I dati utenti sono persistiti correttamente in PostgreSQL.
- La documentazione API è disponibile e descrive tutti gli endpoint previsti.
- Test automatici coprono i casi principali di registrazione, login, accesso protetto e reset password.
- Tutte le password sono salvate in forma hashed in database (mai in chiaro).

## 8. Rischi e punti aperti
- Mancanza di dettagli sul flusso di reset password (es. invio email vs token temporaneo interno).
- Non è definito il tempo di scadenza dei token JWT né le politiche di invalidazione.
- Non sono specificati requisiti di sicurezza avanzata (rate limiting, captcha, MFA).
- Possibili implicazioni di sicurezza dovute all’assenza di meccanismi di verifica email utente.
- Ambiguità sulla granularità e complessità degli endpoint protetti richiesti (quanti e quali).
- Scelta della libreria per hashing e gestione JWT non definita, importante valutarne la solidità.
- Scalabilità e manutenzione futura del sistema non specificate, da pianificare in fase successiva.