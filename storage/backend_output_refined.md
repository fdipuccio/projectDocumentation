MODULE: BACKEND VERSION: 1
## 1. Obiettivo backend
Progettare e implementare un backend RESTful per la gestione dell’autenticazione utenti, comprensivo di registrazione tramite email e password, login con rilascio di token JWT, protezione di endpoint mediante autenticazione JWT e funzionalità base di reset password con generazione e validazione di token reset. Il sistema deve garantire sicurezza minima conforme allo stack tecnico indicato (FastAPI, PostgreSQL, JWT) e coprire le funzionalità MVP richieste, con una documentazione API chiara e una struttura modulare.

## 2. Assunzioni tecniche
- Email come identificatore unico per l’utente.
- Password memorizzate in forma hashata (bcrypt).
- Token JWT con claims standard (sub, iat, exp) e durata configurabile via variabili ambiente; senza claims personalizzati.
- Reset password implementato tramite token univoco salvato in DB, senza invio email né interfacce avanzate.
- Validazione base degli input (email format, password con lunghezza minima).
- Sistema operante su FastAPI con PostgreSQL come database relazionale.
- Gestione della configurazione sensibile tramite variabili ambiente (chiave segreta JWT, credenziali DB).
- Nessuna implementazione di MFA, ruoli, MFA, logging avanzato o integrazioni esterne.
- Il sistema si focalizza su API REST e gestione sessione tramite JWT senza revoca o blacklist.
- Timezone dei timestamp gestiti in UTC per coerenza.
- Gestione errori uniforme, con messaggi standardizzati senza esporre dettagli sensibili.

## 3. Architettura backend
L’architettura è modulare a più livelli:
- **API Layer**: definisce gli endpoint RESTFastAPI con rotte distinte per registrazione, login, reset password e endpoint protetti.
- **Application Layer**: implementa la logica di business, come validazione input, hashing password, generazione e verifica token JWT e reset.
- **Data Layer**: tramite SQLAlchemy gestisce modellazione dati utenti, token reset e interazione con PostgreSQL.
- **Security Layer**: contiene i moduli per hashing (bcrypt), gestione e validazione JWT, oltre a dipendenze FastAPI per protezione degli endpoint.
- **Configurazione** gestita centralmente, con parametri sensibili letti da variabili d’ambiente.
- Test e documentazione sono integrati nel processo di sviluppo.

## 4. Moduli e responsabilità
- **models.py**: definizione modello User, ResetPasswordToken con campi email, password_hash, token, scadenza.
- **schemas.py**: Pydantic schema per input/output API.
- **database.py**: inizializza connessione PostgreSQL tramite SQLAlchemy + session management.
- **auth.py**: funzioni di hashing password (bcrypt), generazione/verifica JWT, generazione token reset.
- **dependencies.py**: dipendenze FastAPI per estrarre l’utente autenticato dal token JWT, proteggere endpoint.
- **routers/user.py**: endpoint registrazione, login, reset password (richiesta token e change).
- **routers/protected.py**: esempio endpoint protetto accessibile solo con JWT valido.
- **core/config.py**: gestione configurazioni (durata token, segreto JWT, parametri DB) da env.
- **exceptions.py**: definizioni errori standardizzati per risposte coerenti.
- **main.py**: avvio FastAPI, includendo router e configurazione middleware.
- **tests/**: unitari e integrati per coprire tutti i casi e validazioni di sicurezza.

## 5. API principali
- **POST /register**: registra nuovo utente  
  Input: email, password  
  Output: conferma creazione (senza password)  
  Validazione unica email, password >=8 chars.
  
- **POST /login**: autenticazione utente  
  Input: email, password  
  Output: token JWT (access_token, tipo Bearer, durata configurata)
  
- **POST /reset-password/request**: genera token reset  
  Input: email  
  Output: messaggio conferma (token reset salvato internamente, non inviato via email)
  
- **POST /reset-password/confirm**: modifica password  
  Input: token reset, nuova password  
  Output: conferma modifica password
  
- **GET /protected/example**: esempio endpoint protetto  
  Output: dati utente autenticato  
  Accesso solo con token JWT valido nell’header Authorization.

## 6. Business logic
- Alla registrazione la password è hashata con bcrypt e salvata; verifica unicità email.
- Al login si verifica la password confrontando con hash bcrypt, se ok si genera JWT firmato con durata configurabile e claim sub(user_id).
- Token JWT include claims standard: sub (user id), iat, exp (es. default a 15 minuti).
- Reset password genera token random univoco (UUID4 o token URL-safe) salvato in DB con timestamp UTC e scadenza (es. 1 h).
- Alla conferma reset password si verifica validità token, ne scade uno solo per utente (eventuale invalidazione token precedenti) e si aggiorna la password con hashing.
- Validazioni input applicate tramite Pydantic e logica business.
- Tutti i segreti (es. chiave JWT) sono isolati in config da env e non hardcoded.
- L’accesso endpoint protetti avviene tramite dipendenza JWT che decodifica token e verifica integrità e scadenza.

## 7. Persistenza e integrazioni
- Database PostgreSQL con SQLAlchemy ORM.
- Modello User: id (pk), email (unique), password_hash, created_at (UTC timestamp).
- Modello ResetPasswordToken: id, user_id (fk), token, expiry timestamp UTC.
- Migrazioni gestite con Alembic per versioni coerenti degli schemi.
- Connessione DB poolata e configurata tramite variabili ambiente (host, port, user, password, dbname).
- Nessun sistema esterno integrato per gestione email o MFA, mock/placeholder futuro per invio email.
- Gestione trasparente e sicura delle transazioni DB nei servizi di business logic.

## 8. Autenticazione e autorizzazione
- Autenticazione tramite JWT standard (HS256).
- Chiave segreta JWT configurata da variabile d’ambiente, durata del token configurabile (default 15 minuti).
- JWT firmati senza claims custom, payload minimale per autenticazione.
- Protezione endpoint tramite dipendenza FastAPI che estrae il token Bearer da header Authorization, ne verifica firma, scadenza e payload.
- Reset password gestito tramite token unico salvato in DB, scaduto dopo durata configurata (es. 1 ora), non mandato via email ma restituito in risposta API per MVP.
- Nessun meccanismo di revoca o blacklist per JWT; suggerita roadmap futura.
- Password reset token invalidati dopo l’uso, un token per volta per utente.
- Nessuna autorizzazione avanzata o ruoli implementati.

## 9. Gestione errori
- Errori restituiti con codici HTTP standard:  
  - 400 Bad Request per input non validi.  
  - 401 Unauthorized per token mancanti o invalidi.  
  - 404 Not Found per risorse non esistenti (es. email non registrata per reset).  
  - 409 Conflict per email già esistente in registrazione.
- Messaggi errori anonimi e non dettagliati per login per non rivelare se email o password sbagliate.
- Risposte JSON standardizzate con struttura: `{ "detail": "messaggio errore" }`.
- Errori interni 500 gestiti a livello globale tramite middleware FastAPI.
- Log errori per debug ma senza esporre dati sensibili nel body di risposta JSON.
- Validazione input Pydantic con gestione coerente degli errori validation_error 422.

## 10. Strategia di test backend
- **Unit test** per:  
  - hashing/validazione password bcrypt.  
  - generazione e verifica JWT (inclusa scadenza).  
  - generazione, salvataggio e validazione token reset.
  
- **Test integrati** per endpoint:  
  - Registrazione: casi validi, email duplicata, password troppo corta, email non valida.  
  - Login: success/failure, token JWT generato e valido.  
  - Endpoint protetto: accesso con token valido, scaduto, mancante, malformato.  
  - Reset password: richiesta token con email valida/invalida, modifica con token valido/scaduto/errato.
  
- **Test sicurezza**:  
  - Verifica password mai restituita nelle API.  
  - Verifica firma e durata token JWT.  
  - Simulazione attacchi replay su token reset.  
  - Concorrenza su registrazione e login.
  
- **Test di conformità** con la documentazione OpenAPI generata da FastAPI.
- Test negativi su formati input non validi e messaggistica errori coerente.
- Copertura minima dei casi edge indicati nel feedback QA.

## 11. Rischi tecnici
- Mancanza di meccanismi per revoca token JWT potrebbe compromettere sicurezza in caso di furto.
- Esposizione del token reset password nella risposta API senza canale sicuro può rappresentare rischio se non protetto da livello superiore.
- Possibile abuso brute force su login/reset password senza rate limiting.
- Gestione timestamps e timezone deve essere rigorosa per evitare incongruenze su scadenza token.
- Mancanza di logging e auditing limita capacità investigazione incidenti; pianificare estensione futura.
- Ambiguità nel formato e messaggi di errore potrebbe influenzare UX e sicurezza; standardizzazione essenziale.
- Dipendenza da configurazione ambiente comporta rischio di errata configurazione in produzione.
- Validazione password minimale riduce la robustezza delle credenziali.
- Possibili condizioni di gara su reset token e concorrenza non gestite esplicitamente.

## 12. Struttura file proposta
```
/app
  /core
    config.py            # configurazione ambiente e parametri
  /models
    models.py            # modelli SQLAlchemy (User, ResetPasswordToken)
  /schemas
    schemas.py           # Pydantic schema per input/output API
  /database
    database.py          # inizializzazione DB, sessioni SQLAlchemy
  /security
    auth.py              # gestione hashing, JWT, reset token
    dependencies.py      # dipendenze FastAPI per protezione endpoint
  /routers
    user.py              # registrazione, login, reset password
    protected.py         # endpoint protetti di esempio
  /exceptions
    exceptions.py        # definizione errori personalizzati API
  /tests
    test_auth.py         # unit e integrati per auth e reset
    test_routes.py       # test integrati endpoint
  main.py                # avvio FastAPI, routing e middleware
alembic/                  # migrazioni DB
.env                      # file variabili ambiente (non versionato)
```

## 13. Piano di implementazione
1. **Setup ambiente e DB**: preparare FastAPI, connettere PostgreSQL, inizializzare ORM, migrazioni Alembic.
2. **Modelli dati**: definire User e ResetPasswordToken, creare prima migrazione.
3. **Configurazione**: definire core/config con variabili ambiente per JWT e DB.
4. **Sicurezza**: implementare hashing password (bcrypt), generazione/verifica JWT, gestione token reset.
5. **Endpoint registrazione**: validazione input, hash password, gestire email uniche, restituire risposta.
6. **Endpoint login**: verifica credentials, generazione JWT, risposta token.
7. **Middleware/dipendenza Auth**: creare dipendenza FastAPI per protezione endpoint con JWT.
8. **Endpoint protetti**: creare endpoint di test autenticati.
9. **Reset password**: generazione token reset, endpoint richiesta e conferma modifica password.
10. **Gestione errori**: implementare risposte standardizzate e gestione eccezioni.
11. **Test**: sviluppare test unitari e integrati per tutte le funzionalità.
12. **Documentazione**: verificare corretta generazione OpenAPI e coerenza endpoint.
13. **Revisione sicurezza**: validare le configurazioni di durata token e sicurezza implementata.
14. **Deployment preparatorio**: preparare scripts/istruzioni per ambiente produzione con gestione configurazione sicura.
15. **Iterazione eventuali migliorie** su gestione configurazioni, messaging e pianificazione futuri upgrade richiesti (rate limiting, revoca token, auditing).