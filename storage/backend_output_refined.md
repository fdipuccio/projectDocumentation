MODULE: BACKEND VERSION: 1
## 1. Obiettivo backend
Progettare e sviluppare un backend API RESTful per la gestione completa dell’autenticazione utenti, includendo la registrazione, login tramite email e password, generazione e validazione di token JWT per la sessione, protezione di endpoint riservati e una funzionalità base di reset password tramite token. Il sistema deve garantire sicurezza minima conforme alle best practice di hashing password e protezione token, con persistenza dati utenti su PostgreSQL, utilizzando il framework FastAPI e il linguaggio Python.

## 2. Assunzioni tecniche
- La persistenza dati sarà garantita tramite PostgreSQL con connessione gestita in modo asincrono o sincrono a seconda della configurazione.
- Password utenti saranno salvate solo in forma hashed (bcrypt o argon2).
- La chiave segreta per la firma dei token JWT sarà configurata esternamente all’applicazione (variabile ambiente) con scadenza configurabile.
- Reset password funziona via token generato e memorizzato nel db, senza invio automatizzato via email nel MVP.
- Endpoints critici implementano validazioni di input di base e restituiscono messaggi di errore non sensibili.
- Non sono implementate rotazione automatica della chiave JWT o blacklist/revoca token nel MVP.
- Nessun sistema di rate limiting integrato nel MVP, ma architettura predisposta per future estensioni.
- L’ambiente di esecuzione prevede containerizzazione o ambiente isolato con accesso diretto a PostgreSQL.

## 3. Architettura backend
L’architettura è modulare e stratificata, separando chiaramente i seguenti livelli:
- **API layer**: definizione e gestione degli endpoint REST su FastAPI, inclusa validazione input/output con Pydantic.
- **Business logic**: gestione della logica applicativa, controllo flussi autenticazione, generazione token, gestione reset password.
- **Persistenza dati**: interazione con PostgreSQL tramite ORM o query dirette, gestione transazioni e modelli dati.
- **Sicurezza**: gestione hashing password, crittografia e firma token JWT, verifica token negli endpoint protetti.
- **Test e documentazione**: una suite di test automatici end-to-end e unitari, documentazione API generata tramite OpenAPI integrata.

Questa architettura garantisce riusabilità, manutenzione facilitata e separazione delle responsabilità, pronta per futuri ampliamenti e integrazioni.

## 4. Moduli e responsabilità
- **models.py**: definizione degli schemi dati utenti e token reset per PostgreSQL.
- **schemas.py**: definizione di modelli Pydantic per validazione API (input/output).
- **database.py**: setup connessione database e sessioni.
- **auth.py**: funzioni di hashing password, generazione/verifica JWT, gestione token reset password.
- **crud.py**: funzioni per creare, leggere, aggiornare e cancellare dati utenti e token nel DB.
- **api/routes/auth.py**: endpoint pubblici per registrazione, login, reset password.
- **api/routes/protected.py**: endpoint protetti tramite dependency di autenticazione JWT.
- **dependencies.py**: funzioni e dependency FastAPI per validare token JWT e proteggere rotte.
- **main.py**: configurazione applicazione FastAPI, inclusione rotte e middleware.
- **tests/**: suite di test automatizzati per tutte le funzionalità chiave.

## 5. API principali
- **POST /register**: registrazione utente con email e password, restituisce messaggio di successo o errore.
- **POST /login**: verifica credenziali utente, restituisce token JWT con payload {user_id, email} e scadenza.
- **POST /reset-password/request**: richiede un token di reset password associato all’email utente, memorizza token e expiry nel DB.
- **POST /reset-password/confirm**: conferma reset password cambiando la password tramite token reset valido.
- **GET /protected/example**: esempio di endpoint protetto, accessibile solo con token JWT valido, restituisce dati dimostrativi.

## 6. Business logic
- Validazione di dati in ingresso con Pydantic e logica di controllo duplicazione email.
- Hashing password utente in fase di registrazione e aggiornamento password reset.
- Autenticazione tramite verifica hash password e generazione token JWT firmato con payload minimi.
- Verifica token JWT in dependency FastAPI per protezione endpoints riservati.
- Generazione token reset password con UUID4 o token sicuro, salvati con timestamp di scadenza in DB.
- Validazione token reset al momento del cambio password e invalidazione del token dopo uso.
- Gestione errori specifici (es. email duplicata, token scaduto/non valido, credenziali errate).

## 7. Persistenza e integrazioni
- PostgreSQL come database relazionale con tabella utenti contenente i campi: id, email univoca, password_hash, reset_token nullable, reset_token_expiry nullable, timestamp creazione/aggiornamento.
- Accesso DB mediato tramite SQLAlchemy ORM o asyncorm per astrazione dati e facilitare test.
- Nessuna integrazione esterna prevista per mailing o altri servizi nel MVP.
- Possibilità di estendere schema dati per future funzionalità (es. ruoli, log accessi).

## 8. Autenticazione e autorizzazione
- Token JWT firmati con una chiave segreta mantenuta in ambiente sicuro, payload minimo contenente user_id ed email.
- Durata del token JWT parametrizzata (es. 15-60 minuti configurabili).
- Validazione e decodifica token tramite libreria standard JWT (es. PyJWT o python-jose).
- Dependency FastAPI per estrazione e verifica token da header Authorization: Bearer, con raising HTTP 401 in caso di invalidità.
- Reset password gestito tramite token specifico memorizzato in DB con scadenza, utilizzabile per cambio password.
- Mancata implementazione di logout o blacklist JWT nel MVP.

## 9. Gestione errori
- Risposte HTTP con codici appropriati:
  - 400 Bad Request per input non validi o dati mancanti.
  - 401 Unauthorized per mancanza o invalidità token JWT.
  - 404 Not Found per utenti o token reset non esistenti.
  - 409 Conflict per email già registrata.
  - 500 Internal Server Error per errori inaspettati.
- Messaggi di errore non rivelano informazioni sensibili (es. dettagli sull’hashing o struttura interna).
- Gestione centralizzata delle eccezioni tramite FastAPI Exception Handlers.
- Logging degli errori critici per audit e troubleshooting, con informazioni minimali sugli errori lato client e dettagli completi lato server.

## 10. Strategia di test backend
- Test automatici unitari e di integrazione per tutte le API endpoint critici e logica business.
- Casi di test per registrazione: email valida, duplicata, formati errati, password debole.
- Casi di test per login: credenziali corrette, errate, utenti non esistenti.
- Test accesso endpoint protetti con token valido, scaduto, non valido, mancante.
- Test reset password: richiesta token con utenti esistenti e inesistenti, conferma reset con token valido, scaduto, non valido.
- Test input validation per tutte le richieste.
- Test error handling verificando risposte HTTP e messaggi corretti.
- Test di concorrenza semplice per generazione token reset.
- Test end-to-end con database PostgreSQL in ambiente isolato.
- Possibilità di integrazione futura con test di sicurezza e pen test.

## 11. Rischi tecnici
- Assenza di rotazione automatica e protezione avanzata chiave JWT può esporre a rischio compromissione a lungo termine.
- Mancanza meccanismo di revoca o blacklist token JWT limita controllo su sessioni e logout forzato.
- Reset password via token token restituito direttamente nell’API senza invio email crea potenziali rischi di abuso in mancanza di controllo accessi.
- Mancanza rate limiting espone a potenziali attacchi brute force su login e reset.
- Log insufficienti per auditing sicurezza limitano capacità di monitorare anomalie o accessi malevoli.
- Mancanza di policy password robuste potrebbe consentire password deboli.
- Architettura monolitica per gestione token reset può generare problemi scalabilità e concorrenza.
- Nessun alerting o monitoraggio automatico integrato lato produzione.
- Richiesta coordinamento con team di sicurezza per ampliamento policy future e hardening.

## 12. Struttura file proposta
```
/app
 ├── main.py                  # Entry point, configurazione FastAPI
 ├── database.py              # Connessione e setup DB PostgreSQL
 ├── models.py                # ORM models per utenti e token reset
 ├── schemas.py               # Pydantic schemi input/output API
 ├── auth.py                  # Gestione hashing, JWT, token reset
 ├── crud.py                  # Operazioni CRUD DB
 ├── dependencies.py          # Dependency FastAPI per autenticazione
 ├── api
 │    ├── __init__.py
 │    ├── routes
 │    │    ├── auth.py        # Endpoints registrazione, login, reset password
 │    │    └── protected.py   # Endpoint protetti esempi
 └── tests
      ├── test_registration.py
      ├── test_login.py
      ├── test_protected_endpoints.py
      ├── test_reset_password.py
      └── conftest.py         # Setup test database, fixtures
```

## 13. Piano di implementazione
1. Progettazione schemi DB utenti con campi richiesti e migrazioni DB.
2. Setup ambiente FastAPI con connessione PostgreSQL e configurazione base.
3. Implementazione hashing password e funzioni JWT in modulo auth.py.
4. Sviluppo endpoint /register con validazione e CRUD per inserimento utenti.
5. Sviluppo endpoint /login con verifica credenziali, generazione token JWT.
6. Creazione dependency FastAPI per autenticazione con verifica JWT.
7. Implementazione endpoint protetto di esempio /protected/example.
8. Sviluppo funzionalità reset password:
   - Endpoint richiesta token reset (generate e salva token e expiry).
   - Endpoint conferma reset con token e cambio password.
9. Scrittura test automatici per tutte le funzionalità core con database isolato.
10. Documentazione API generata automaticamente e revisione manuale contenuti.
11. Implementazione gestione errori centralizzata e logging.
12. Review sicurezza, policy chiave JWT, e pianificazione di miglioramenti futuri.
13. Deployment in ambiente di test e validazione finale.