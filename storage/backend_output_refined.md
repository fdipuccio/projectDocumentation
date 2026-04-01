MODULE: BACKEND VERSION: 1
## 1. Obiettivo backend
Sviluppare un backend API REST per la gestione dell’autenticazione utenti, fornendo funzionalità di registrazione, login tramite email e password, generazione token JWT per autenticazione protetta, protezione di endpoint tramite validazione del token, e un flusso base di reset password. Il sistema deve garantire sicurezza basica enterprise (es. hashing password sicuro) e struttura modulare e riusabile per futuri ampliamenti, coerente con lo stack FastAPI, PostgreSQL e JWT.

## 2. Assunzioni tecniche
- Flusso di reset password è "base": invio di token tramite email previsto da altro team, con scadenza token gestita lato backend.
- Password memorizzata in forma hashata con bcrypt o equivalente.
- JWT utilizzato con algoritmo standard (es. HS256), payload minimalista e scadenza memoria sia consigliata che configurabile.
- Nessuna gestione avanzata di ruoli, MFA, autenticazione social o policy password complessa (ma progettato per futuri ampliamenti).
- Configurazione segreti e parametri (JWT secret, DB credentials, scadenze token) tramite variabili ambiente per gestione multi-ambiente.
- Niente gestione diretta di invio email, logging avanzato o controlli di sicurezza extra (rate limiting) nell’MVP.
- Sistema predisposto per ambienti enterprise con possibilità di evoluzione, ma senza caratteristiche di scalabilità o alta disponibilità previste.

## 3. Architettura backend
- Architettura a più livelli con separazione chiara tra API layer (FastAPI), business logic (servizi di gestione utenti, autenticazione, token), e accesso a dati tramite repository.
- Moduli isolati per logging, sicurezza e configurazione.
- Uso di FastAPI come framework per routing, validazione dati e generazione OpenAPI/Swagger.
- Connessione a PostgreSQL mediante ORM (ad es. SQLAlchemy o equivalente leggero) per gestione entità utenti.
- Dipendenze FastAPI per gestione autenticazione JWT su endpoint protetti.
- Meccanismo di gestione configurazione da variabili ambiente, con supporto per override in ambienti diversi (sviluppo, staging, produzione).

## 4. Moduli e responsabilità
- **modello_dati**: definizione schema utenti (email, password hash, token reset, timestamp creazione e aggiornamento, scadenza token reset).
- **database**: gestione connessione e sessioni PostgreSQL.
- **servizi_utente**: logica di registrazione, verifica credenziali, generazione token reset, aggiornamento password.
- **autenticazione**: generazione/validazione token JWT, dipendenza FastAPI per protezione endpoint.
- **api_routes**: definizione endpoint REST (registrazione, login, reset password, test endpoint protetti).
- **configurazione**: gestione configurazioni/caricamenti variabili ambiente.
- **test**: test unitari e integrativi per flussi principali e casi limite.
- **utils**: funzioni comuni per hashing (bcrypt), gestione errori e validazioni base.

## 5. API principali
- `POST /register`  
  - Registra nuovo utente. Input: email, password.  
  - Validazione email unica, password hashata.  
  - Risposta: stato creazione utente o errore (409 se email esistente).  

- `POST /login`  
  - Login utente. Input: email, password.  
  - Verifica password hashata, restituisce token JWT con scadenza.  
  - Risposta: token JWT o errore 401.  

- `POST /password-reset/request`  
  - Richiede reset password. Input: email.  
  - Genera token reset con scadenza, memorizza in DB. Invio email esterno.  
  - Risposta: conferma richiesta inviata (indipendentemente email esistente).  

- `POST /password-reset/confirm`  
  - Conferma reset password. Input: token reset, nuova password.  
  - Verifica validità token e scadenza, aggiorna password (hashata).  
  - Risposta: conferma o errore (token invalido/scaduto).  

- `GET /protected/resource`  
  - Endpoint protetto per test autenticazione. Accesso solo con JWT valido.  
  - Risposta: dati protetti o errore 401 se token assente o invalido.

## 6. Business logic
- Registrazione verifica email univoca, valida formato base, hashes password con bcrypt.  
- Login confronta password hashed, genera JWT contenente user id e scadenza (configurabile).  
- Reset password token è UUID o token sufficientemente casuale salvato col timestamp di scadenza (es. 1 ora).  
- Validazioni base sugli input, gestendo errori coerenti.  
- Protezione endpoint tramite dipendenza FastAPI che decodifica e valida JWT, rifiuta richieste con token non valido o scaduto.  
- Logging di base per errori e richieste critiche (implementabile da estendere).  
- Possibilità di introdurre futuri miglioramenti per policy password, gestione ruoli, MFA.

## 7. Persistenza e integrazioni
- PostgreSQL come database relazionale principale: tabella utenti con colonne id, email (unique), password_hash, reset_token, reset_token_expiry, created_at, updated_at.  
- Connessione tramite pool gestito da ORM (SQLAlchemy o equivalente).  
- Nessuna integrazione diretta per invio email (previsto da altri team via sistema separato).  
- Tutti gli accessi DB passano tramite moduli dedicati per facilitare testing e mantenibilità.  
- Persistenza token reset con scadenza per sicurezza di base.

## 8. Autenticazione e autorizzazione
- Password memorizzata in modo sicuro col metodo bcrypt (salt incluso).  
- Token JWT firmato con secret configurabile, algoritmo HS256, payload contenente user_id e scadenza (es. 15-60 minuti, configurabile).  
- Dipendenza FastAPI per verifica e decoding JWT su chiamate protette, generazione di 401 in caso di mancata validazione.  
- Reset password tramite token temporaneo salvato in DB con scadenza.  
- Non prevista gestione ruoli o permessi avanzati nell’MVP.

## 9. Gestione errori
- Codici HTTP coerenti: 201 per creazione avvenuta, 400 per input non valido, 401 per errori autenticazione, 404 o 409 in caso di risorse non trovate o conflitti (es. email duplicata).  
- Messaggi di errore informativi ma non eccessivamente dettagliati per non rivelare vulnerabilità.  
- Gestione centralizzata di errori tramite exception handlers FastAPI.  
- Logging errori di sistema per facilitarne audit successivi.  
- Endpoint reset password confermano richiesta indipendentemente dall’esistenza dell’email per non rivelare informazioni utenti.

## 10. Strategia di test backend
- Test unitari per servizi utenti: registra, login, hashing password, generazione/validazione token reset.  
- Test integrativi endpoint API per flussi completi: registrazione, login con success/failure, reset password (request e confirm) con token valido, invalido e scaduto.  
- Test endpoint protetti con JWT valido e non valido (token mancante, malformato, scaduto).  
- Test validazione input per prevenzione injection o formati email/pwd errati.  
- Test negativi e edge case (email duplicata, token reset errato, password vuota).  
- Verifica sicurezza hashing password (non memorizzata in chiaro).  
- Verifica corretta generazione e decoding JWT (payload e firma).  
- Stress test base per simulazione carico e controllo stabilità (facoltativo, suggerito per futuri upgrade).  

## 11. Rischi tecnici
- Ambiguità e debolezza del flusso reset password base, in mancanza di integrazione email immediata ed eventuale mancata scadenza token.  
- Mancanza di policy obbligatorie per password robuste può esporre utenti a rischi (da valutare evoluzione).  
- Assenza di gestione di scadenza rigida dei token JWT e rinnovi possono esporre a sessioni prolungate non sicure.  
- Nessuna misura anti brute force/rate limiting su login e reset password vulnerabilizza sistema.  
- Future richieste di MFA, ruoli, o social login potrebbero richiedere significativa rifattorizzazione architetturale.  
- Mancanza di gestione multi-ambiente formalizzata e logging avanzato limita audit e operazioni enterprise.  
- Assenza di backup e disaster recovery previsto nel MVP che andrà pianificato esternamente.  

## 12. Struttura file proposta
```
/app
  /api
    __init__.py
    routes_auth.py            # Registrazione, login, reset password
    routes_protected.py       # Endpoint protetti di test e futuri
  /core
    __init__.py
    config.py                 # Gestione configurazione/variabili ambiente
    security.py               # Hashing password, JWT token utils
    exceptions.py             # Classi eccezioni custom e handler
  /db
    __init__.py
    base.py                   # Connessione e sessione database
    models.py                 # Modello dati utenti
    repository.py             # Accesso dati (query utenti e token)
  /services
    __init__.py
    user_service.py           # Logica business utente (registrazione, login, reset)
    auth_service.py           # JWT generazione, validazione token
  /tests
    test_register.py
    test_login.py
    test_password_reset.py
    test_auth_protected.py
main.py                      # FastAPI app entry point e mounted route includings
.env                        # File con variabili ambiente (non in repository)
/requirements.txt            # Dipendenze python
```
## 13. Piano di implementazione
1. Progettazione schema dati utenti e creazione migrazione DB (email unica, password hashata, token reset con scadenza).  
2. Configurazione ambiente FastAPI e connessione PostgreSQL con ORM.  
3. Implementazione logica hashing password e gestione configurazioni sicure.  
4. Implementazione endpoint registrazione con validazione base.  
5. Implementazione login verificando password e generando token JWT con scadenza definita.  
6. Implementazione dipendenza/middleware FastAPI per proteggere endpoint con validazione JWT.  
7. Implementazione endpoint reset password: richiesta token, memorizzazione con scadenza, conferma reset password con token.  
8. Scrittura test unitari e integrativi per tutti i flussi critici (registrazione, login, reset password, endpoint protetti).  
9. Generazione automatica documentazione API (OpenAPI/Swagger) con FastAPI.  
10. Revisione sicurezza base: hashing saltato, gestione segreti via ambiente, controllo errori HTTP.  
11. Integrazione e collaudo end-to-end, raccolta feedback, estensioni future (policy password, logging, multipli ambienti).  
12. Documentazione completata con indicazioni per utilizzo e ampliamenti futuri.