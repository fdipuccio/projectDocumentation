MODULE: PM VERSION: 1
## 1. Obiettivo
Sviluppare un backend per un sistema di autenticazione utenti che consenta la registrazione, il login tramite email e password, la generazione di token JWT per sessioni sicure, la protezione di endpoint riservati e una funzionalità base di reset password, utilizzando tecnologie Python, FastAPI e PostgreSQL.

## 2. Contesto e vincoli
- Il backend deve essere implementato usando FastAPI come framework web.
- Il database di persistenza deve essere PostgreSQL.
- L'autenticazione deve utilizzare token JWT per la gestione delle sessioni.
- Il sistema deve prevedere endpoint protetti che richiedano autenticazione valida.
- La funzionalità di reset password deve avere un livello base di implementazione (non ulteriormente specificata).
- Stack tecnologico definito: Python, FastAPI, PostgreSQL, JWT.

## 3. Assunzioni
- La gestione dei dati utenti (es. password) sarà conforme alle best practice di sicurezza (es. hashing sicuro), anche se non esplicitato.
- L’invio di email per reset password o altre funzionalità non è specificato, quindi non incluso nel MVP.
- I token JWT saranno firmati con una chiave segreta gestita nel backend, ma dettagli su rotazione o scadenza sono da definire.
- Il reset password base implica una funzionalità minimale (ad esempio reset tramite endpoint con token), senza interfacce complesse né mailing automatico.
- Il database PostgreSQL è già predisposto e accessibile dal backend.

## 4. Scope MVP
- Implementazione dell’API di registrazione utente con validazione base dei dati.
- Implementazione di login con verifica email e password, restituzione di token JWT.
- Generazione e gestione di token JWT per autenticazione e autorizzazione.
- Creazione di endpoint protetti da autenticazione (es. endpoint esempio protetto).
- Implementazione base del reset password (es. richiesta reset e cambio password tramite token).
- Persistenza dati utenti e token nel database PostgreSQL.

## 5. Out of scope
- Gestione avanzata di sicurezza (es. brute force protection, rate limiting).
- Integrazione con servizi esterni di posta elettronica.
- Interfacce utente o front-end.
- Funzionalità avanzate di reset password (es. invio link via email, MFA).
- Gestione di ruoli/permessi complessi oltre il semplice accesso autenticato.

## 6. Task tecnici ordinati
1. Progettazione dello schema database utenti con campi essenziali (email, password hash, reset token, ecc.).
2. Setup ambiente FastAPI con configurazione base e connessione PostgreSQL.
3. Implementazione endpoint registrazione con validazione input e salvataggio sicuro password.
4. Implementazione endpoint login che verifica credenziali, genera e restituisce token JWT.
5. Realizzazione middleware/depends per verifica token JWT e protezione endpoint.
6. Implementazione endpoint protetto di esempio per validazione accesso autenticato.
7. Sviluppo funzionalità base reset password: generazione token reset, endpoint per cambio password con token.
8. Scrittura di test funzionali su registrazione, login, protezione endpoint e reset password.
9. Documentazione API base (ad es. OpenAPI/Swagger).

## 7. Acceptance criteria
- È possibile registrare un nuovo utente con email e password.
- Un utente registrato può effettuare il login e ricevere un token JWT valido.
- Gli endpoint protetti sono accessibili solo con token JWT valido.
- È possibile richiedere un reset password via API e cambiare la password utilizzando il token di reset.
- I dati utente sono salvati correttamente nel database PostgreSQL.
- Tutte le funzionalità rispettano i vincoli tecnologici (FastAPI e PostgreSQL).
- Sono presenti test automatici che confermano il funzionamento delle funzionalità principali.

## 8. Rischi e punti aperti
- Mancanza di dettagli su come trattare la funzionalità reset password (es. invio token, scadenza token).
- Nessuna indicazione su requisiti di sicurezza specifici (es. complessità password, cifratura chiave JWT).
- Possibili problemi di gestione della scalabilità o performance non considerati nel MVP.
- Non è definito il formato o il payload esatto del token JWT.
- Assenza di requisiti su gestione sessioni multiple o invalidazione token.
- Necessità di coordinamento con team di sicurezza o altri stakeholder per definire policy più dettagliate.