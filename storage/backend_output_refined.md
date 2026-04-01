MODULE: BACKEND VERSION: 1
## 1. Obiettivo backend
Progettare e implementare un backend API in FastAPI (Python) per un sistema di autenticazione utenti che consenta la registrazione con email e password, il login con generazione e validazione di token JWT per sessioni sicure, la protezione degli endpoint tramite autenticazione JWT e una gestione base del reset password con token temporanei. Il sistema deve garantire la persistenza sicura dei dati utenti e dei token su PostgreSQL, fornire endpoint REST conformi ai requisiti e generare documentazione OpenAPI accessibile.

## 2. Assunzioni tecniche
- Il backend utilizza Python con il framework FastAPI e PostgreSQL come database relazionale.
- Le password sono memorizzate in forma hashed usando l’algoritmo bcrypt.
- I token JWT sono generati e verificati secondo standard HS256 o RS256, con durata configurabile (durata base es. 15-60 minuti).
- Non sono previsti meccanismi di gestione ruoli, multi-fattore o invio email/SMS; il reset password è basico e senza notifica esterna.
- L’ambiente di deploy gestisce aspetti infrastrutturali esterni (SSL, CORS, load balancing).
- L’integrazione con stack Java REST API esistente è da definire esternamente e non impatta la soluzione FastAPI, che si propone come modulo indipendente.
- Validazione d’input e gestione casi di errori comuni (es. email duplicata, input non valido) sono previsti nelle API.
- Il token di reset password è temporaneo, monouso e memorizzato nel database associato all’utente.
- La documentazione OpenAPI sarà generata automaticamente da FastAPI.

## 3. Architettura backend
L’architettura è modulare e pulita, secondo il modello tipico FastAPI:
- Layer di presentazione con le route API REST e gestione delle request/response.
- Service layer contenente la business logic relativa all’autenticazione, gestione utenti e reset password.
- Data access layer implementato tramite ORM SQLAlchemy per interagire con PostgreSQL.
- Moduli di sicurezza per hashing password, gestione e verifica token JWT, e protezione degli endpoint.
- Componenti di utilità e configurazione per gestione variabili ambientali, segreti e setup database.
- Middleware/Dependency Injection FastAPI per la protezione di endpoint tramite autenticazione JWT.
- Sistema di test unitari e integrazione automatizzati.
  
Questa separazione consente facile manutenibilità, testabilità e estendibilità del codice.

## 4. Moduli e responsabilità
- **models.py**: definizione dei modelli dati ORM, inclusi User e TokenResetPassword.
- **schemas.py**: definizione degli schemi Pydantic per validazione dati in input/output API.
- **database.py**: configurazione della connessione e sessione al database PostgreSQL.
- **auth.py**: gestione hashing password, generazione/verifica token JWT, funzioni di autenticazione.
- **crud.py**: funzioni per CRUD utenti e token reset password nel database.
- **api/routes/auth.py**: endpoints di registrazione, login, logout (se applicabile).
- **api/routes/user.py**: endpoint protetti, reset password, user info.
- **dependencies.py**: dipendenze FastAPI per autenticazione tramite JWT.
- **main.py**: entry point FastAPI, include montaggio router, configurazioni principali e middleware.
- **tests/**: test unitari e di integrazione coprenti tutte le funzionalità principali e casi di errore.
- **config.py**: gestione configurazioni ambientali e segreti (JWT secret, durata token).

## 5. API principali
- **POST /auth/register**: registrazione utente con email e password; valida input e gestisce utenti duplicati.
- **POST /auth/login**: login con email e password; restituisce token JWT firmato.
- **POST /auth/reset-password/request**: genera token temporaneo per reset password associato all’email fornita; senza invio email.
- **POST /auth/reset-password/confirm**: conferma reset password fornendo token reset e nuova password, valida token e aggiorna password.
- **GET /user/me**: endpoint protetto che ritorna informazioni dell’utente autenticato.
- Altri endpoint protetti possono essere aggiunti estendendo lo schema sopra.

Tutti gli endpoint protetti richiedono header `Authorization: Bearer <token JWT>`.

## 6. Business logic
- Registrazione: verifica unicità email, validazione password (es. lunghezza minima), hashing password con bcrypt e salvataggio utente.
- Login: verifica esistenza email, confronto password hashed, generazione token JWT con payload contenente user id e scadenza.
- Protezione endpoint: middleware verifica validità token JWT, estrae utente e autorizza accesso.
- Reset password: generazione token univoco (UUID o JWT con scadenza breve), salvataggio token accoppiato a utente, validazione token al reset, password hashing e update.
- Gestione errori coerente con standard HTTP (400 Bad Request, 401 Unauthorized, 409 Conflict per duplicate email).
- Non è prevista la revoca token JWT in tempo reale; invalidazione indiretta gestita dalla scadenza o cambi password.

## 7. Persistenza e integrazioni
- PostgreSQL per persistenza dati utenti e token reset password.
- ORM SQLAlchemy per astrazione query, con sessioni transazionali, uso di migration (es. Alembic) per evoluzioni schema database.
- Nessuna integrazione esterna (email, SMS) per reset password o notifiche.
- Gestione configurazione connessione DB tramite variabili ambiente.
- Persistenza sicura password hashed, token reset password memorizzati solo temporaneamente con data scadenza.

## 8. Autenticazione e autorizzazione
- Autenticazione basata su JWT: token generato con payload minimale (user id, scadenza), firmato con chiave segreta HS256 o coppia RS256.
- Token JWT valido richiesto per accesso endpoint protetti tramite apposite dipendenze in FastAPI.
- Hashing password con bcrypt per sicurezza.
- Reset password con token temporaneo e univoco memorizzato e associato a utente; scaduto dopo breve tempo (es. 1 ora).
- Non è prevista gestione ruoli o permessi avanzati; autorizzazione limitata a controllo presenza e validità token autenticazione.

## 9. Gestione errori
- Validazione input con Pydantic e gestione errori 400 per input errati o mancanti.
- 401 Unauthorized per chiamate a endpoint protetti senza o con token non valido/scaduto.
- 409 Conflict per tentativi di registrazione con email già esistente.
- Risposte strutturate in JSON con dettagli errore standardizzati.
- Logging minimo in modalità sviluppo per errori critici; logging più avanzato e monitoraggio esterni fuori scope.
- Gestione eccezioni DB con rollback per mantenere consistenza dati.

## 10. Strategia di test backend
- Test unitari su funzioni di hashing, generazione e verifica token JWT, validazione dati.
- Test integrazione endpoint REST simulando flussi completi: registrazione, login, accesso protetto, reset password.
- Casi edge testati: email duplicata, password non valida, token reset scaduto o inesistente, token JWT alterato o mancante.
- Test sicurezza base per robustezza password hashing e corretto payload token JWT.
- Test accesso endpoint protetti senza token o con token invalidi/scaduti.
- Esecuzione test con framework pytest e FastAPI TestClient.
- Progressive integrazione nel CI/CD previsto dal team infrastrutture.

## 11. Rischi tecnici
- Ambiguità inerente coexistence o sostituzione stack Java REST API; possibile complessità di integrazione o deploy multipli non gestita.
- Funzionalità reset password senza invio email limita l’efficacia nella pratica operativa.
- Mancanza di politiche di sicurezza avanzate (es. lockout, rate limiting) espongono rischio di attacchi brute force.
- Gestione segreti JWT e loro protezione non dettagliata, potenziali vulnerabilità se non curata correttamente.
- Assenza di meccanismo di revoca token JWT attiva può creare rischi in caso di compromissione account.
- Elevata importanza di coordinamento con team infrastrutture per configurazioni ambientali, backup e scaling.
- Possibile necessità futura di estendere sistema per gestione ruoli e autorizzazioni avanzate.

## 12. Struttura file proposta
```
/app
  /api
    /routes
      auth.py
      user.py
  /core
    config.py
    security.py
  /crud
    user.py
    reset_password.py
  /db
    database.py
    models.py
  /schemas
    user.py
    token.py
  /tests
    test_auth.py
    test_user.py
    test_reset_password.py
  dependencies.py
  main.py
```
- `/api/routes`: definizione endpoint REST.
- `/core`: configurazioni e funzioni sicurezza (hashing, JWT).
- `/crud`: operazioni DB astratte.
- `/db`: modelli ORM e connessione DB.
- `/schemas`: Pydantic schemas.
- `/tests`: test unitari e integrazione.
- `dependencies.py`: funzioni per autenticazione tramite FastAPI dependency.
- `main.py`: entry point app FastAPI.

## 13. Piano di implementazione
1. Setup ambiente FastAPI e connessione a PostgreSQL con ORM SQLAlchemy.
2. Definizione modelli dati (User, TokenResetPassword) e migrazioni DB.
3. Implementazione endpoint registrazione /auth/register con validazione e gestione duplicati.
4. Implementazione endpoint login /auth/login con verifica credenziali e generazione JWT.
5. Sviluppo modulo gestione token JWT: firma, verifica, durata configurabile.
6. Implementazione dipendenza FastAPI per protezione endpoint (middleware JWT).
7. Creazione endpoint protetto esempio /user/me che ritorna info utente autenticato.
8. Implementazione endpoint reset password /auth/reset-password/request e /auth/reset-password/confirm con token temporaneo.
9. Testing completo unitario e integrazione su tutti i moduli.
10. Generazione automatica documentazione OpenAPI/Swagger visibile su /docs.
11. Revisione sicurezza: hash password, gestione segreti JWT, politica durata token.
12. Revisione codice con peer review focalizzata su sicurezza e robustezza.
13. Coordinamento con team infrastrutture per deploy, configurazione e successiva messa in produzione.

Questa roadmap garantisce una progressione modulare efficiente e la copertura dei requisiti MVP secondo la specifica e feedback QA.