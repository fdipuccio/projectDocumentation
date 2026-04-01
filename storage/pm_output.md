MODULE: PM VERSION: 1
## 1. Obiettivo
Sviluppare un backend sicuro e scalabile per l'autenticazione degli utenti che includa la registrazione, login tramite email e password, generazione di token JWT per la gestione delle sessioni, protezione degli endpoint riservati e una funzionalità base di reset password.

## 2. Contesto e vincoli
- Il backend deve essere sviluppato utilizzando il framework FastAPI.
- Il sistema di persistenza dati dovrà utilizzare PostgreSQL.
- L'autenticazione sarà basata su token JWT per la gestione delle sessioni utente.
- Le funzionalità richieste sono limitate alla registrazione, login, protezione endpoints, generazione token e reset password in modalità base.
- Use Python come linguaggio di programmazione.
- Non sono state fornite indicazioni su sistemi esterni di autenticazione o sicurezza avanzata (es. OAuth, 2FA, CAPTCHA).

## 3. Assunzioni
- La registrazione richiede email univoca e password con requisiti minimi (non specificati, da definire).
- Il reset password "base" sarà implementato tramite invio di email con link/token di reset; tuttavia, la modalità di invio email e template non sono specificati e dovranno essere definiti o utilizzati sistemi esterni esistenti.
- Non è indicata una gestione avanzata delle sessioni o revoca token JWT.
- La protezione degli endpoint riguarda esclusivamente il controllo dell’autenticazione tramite JWT.
- Non sono richiesti ruoli utenti o autorizzazioni granulari.
- Non è specificato il dettaglio dei campi utente oltre email e password.
- La gestione della scalabilità o dei backup del database non è parte del requirement.

## 4. Scope MVP
- Endpoint per registrazione utente con validazione dati.
- Endpoint login email-password che restituisce token JWT.
- Middleware o dipendenza per proteggere endpoint tramite verifica JWT.
- Endpoint base per reset password (invio email con token o link di reset).
- Persistenza utenti e token in PostgreSQL.
- Gestione errori e response standard RESTful.

## 5. Out of scope
- Funzionalità di autorizzazione avanzata (ruoli, permessi).
- Sicurezza avanzata (2FA, CAPTCHA, limitazioni tentativi login).
- Interfacce UI o frontend.
- Integrazione con provider di autenticazione esterni (Google, Facebook, etc).
- Meccanismi di logging specifici o auditing.
- Gestione sessioni complesse o revoca token.
- Scalabilità e performance tuning avanzato.
- Gestione multilingua o internazionalizzazione.

## 6. Task tecnici ordinati
1. Definizione schema dati utenti e tabelle PostgreSQL.
2. Setup ambiente FastAPI con connessione a PostgreSQL.
3. Sviluppo endpoint registrazione utente con hashing password (es. bcrypt).
4. Sviluppo endpoint login autenticazione e generazione token JWT.
5. Implementazione middleware/dipendenza FastAPI per protezione endpoint tramite JWT.
6. Creazione endpoint protetto di esempio per validazione accesso.
7. Implementazione base funzionalità reset password:
   - generazione token di reset temporaneo
   - endpoint invio email reset (configurare SMTP o placeholder)
   - endpoint modifica password con token valido
8. Testing funzionale e sicurezza base (validazione input, error handling).
9. Documentazione API (OpenAPI/Swagger automatico con FastAPI).

## 7. Acceptance criteria
- Utente può registrarsi fornendo email e password; email deve essere unica.
- Utente può autenticarsi con email e password validi, ricevendo token JWT.
- Endpoint protetti restituiscono 401/403 se token JWT assente o non valido.
- Funzionalità di reset password consente il recupero tramite email e modifica password.
- Il database PostgreSQL è correttamente configurato e mantiene gli utenti e i token necessari.
- Tutte le API sono documentate e testate con casi positivi e negativi.
- Il codice segue best practice di sicurezza basilari (hash password, validazione input).

## 8. Rischi e punti aperti
- Mancanza di dettagli sul livello di sicurezza richiesto per password e reset password potrebbe portare a implementazioni poco robuste.
- Meccanismo di invio email per reset password non definito, potrebbe richiedere integrazione con servizi esterni.
- Ambiguità nella gestione scadenza e revoca token JWT.
- Non è specificato se il sistema debba supportare campi utente aggiuntivi o profili estesi.
- Possibili integrazioni future con sistemi di autenticazione esterni o sistemi di autorizzazioni non previste ora.
- Necessità di definire standard di password e policy sicurezza.