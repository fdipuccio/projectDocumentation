MODULE: BACKEND VERSION: 1
## 1. Obiettivo backend
Progettare e sviluppare un backend API in FastAPI per la gestione dell’autenticazione utenti in ambito web/mobile, che consenta la registrazione tramite email e password, login con generazione di token JWT per autenticazione stateless, protezione di endpoint sensibili tramite verifica token e una funzionalità base di reset password con token temporanei. Tutto questo utilizzando PostgreSQL come sistema di persistenza dati, garantendo una soluzione modulare, sicura nei requisiti base, sufficientemente testata e documentata tramite OpenAPI.

## 2. Assunzioni tecniche
- Le password verranno salvate dopo hashing sicuro tramite bcrypt.
- L’email è il campo univoco per identificare l’utente nel DB.
- Il token JWT conterrà l’ID utente e la scadenza configurabile, firmato con algoritmi sicuri (es. HS256).
- I token reset per recupero password sono token temporanei generati e memorizzati nel DB, senza integrazione email o canali esterni.
- Tutti gli accessi protetti richiedono header Authorization con Bearer token JWT valido e non scaduto.
- La complessità della password sarà minima ma con validazione formale di base per evitare input banali.
- Configurazioni sensibili (JWT secret, connessione DB, scadenze token) saranno lette da variabili ambiente e isolate.
- Non è prevista gestione di sessioni multiple, revoca token JWT o logging/audit avanzato in questo MVP.
- Le richieste di reset password e login non implementano throttling o protezioni anti brute force, da valutare in evoluzione.

## 3. Architettura backend
Architettura REST API stateless realizzata con FastAPI, organizzata in moduli chiari: API routes, servizi gestionali (business logic), modelli dati (ORM SQLAlchemy), e meccanismi di sicurezza (JWT, hashing). Il backend interagisce con PostgreSQL tramite SQLAlchemy ORM e gestisce lo schema con Alembic per migrazioni versionate. I flussi di autenticazione sono separati dalla logica di persistente. La validazione dati è effettuata tramite Pydantic. La configurazione è centralizzata e carica parametri da .env/variabili ambiente. La sicurezza JWT è implementata tramite dependency FastAPI come dipendenza per endpoint protetti.

## 4. Moduli e responsabilità
- **models.py**: Definizione modello User (email, hashed_password, reset_token, reset_token_expiry) con SQLAlchemy.
- **schemas.py**: Definizione modelli Pydantic per input/output API (registrazione, login, reset password).
- **database.py**: Setup connessione PostgreSQL, creazione sessioni DB con SQLAlchemy.
- **auth.py**: Funzioni hashing password (bcrypt), generazione/validazione JWT, generazione/validazione token reset.
- **dependencies.py**: Dependency FastAPI per estrazione e validazione JWT da header, verifica utente autenticato.
- **routes/auth.py**: Endpoint registrazione (/register), login (/login), reset password (/reset-password-request e /reset-password).
- **routes/protected.py**: Endpoint protetto di esempio (/protected-resource).
- **config.py**: Caricamento centralizzato configurazione (DB URI, JWT secret, scadenze).
- **tests/**: Test unitari e funzionali per hashing, autenticazione, reset password, protezione endpoint.
- **migrations/**: Migrazioni Alembic per schema utente.

## 5. API principali
- **POST /register**: Registrazione utente, input email e password, validazione formale, salvataggio con hash password.
- **POST /login**: Login utente, verifica password, restituisce token JWT con scadenza.
- **POST /reset-password-request**: Richiesta token reset password via email, genera token temporaneo memorizzato.
- **POST /reset-password**: Reset password fornendo token reset valido e nuova password, aggiorna password hashed.
- **GET /protected-resource**: Endpoint esempio protetto accessibile solo con JWT valido, risorsa demo.
- Tutti gli endpoint espongono risposte strutturate con codice HTTP appropriato (201, 200, 400, 401, etc.) e messaggi di errore chiari.

## 6. Business logic
- Validazione rigida di email (formato RFC standard).
- Validazione password minima: lunghezza, presenza caratteri alfanumerici di base.
- Hashing password con bcrypt e salt strong random.
- Generazione JWT con payload user_id, scadenza configurabile, firma segreta.
- Funzioni per generazione token reset univoco con scadenza breve (es. 1 ora).
- Verifica token JWT all’accesso endpoint, decodifica e validazione firma/scadenza.
- Aggiornamento password solo tramite token reset valido e non scaduto.
- Gestione duplicati email (errore 400 con messaggio chiaro).
- Error handling centralizzato per risposte API coerenti e informative.
- Modularità per facilitare estensioni future (es. refresh token, logging).

## 7. Persistenza e integrazioni
- PostgreSQL come DB relazionale primario.
- SQLAlchemy ORM per mapping oggetti-relazionale.
- Alembic per gestione e migrazione dello schema DB (tabella User con: id PK, email unico, hashed_password, reset_token nullable, reset_token_expiry nullable datetime).
- Connessione DB ottimizzata con connection pooling configurabile.
- Nessuna integrazione esterna (es. email service) inclusa nello scope.
- Gestione connessione stabile con retry/check errori di rete DB.
- Transazioni atomiche su operazioni critiche (registrazione, reset password).

## 8. Autenticazione e autorizzazione
- JWT come meccanismo di autenticazione stateless e autorizzazione per accesso a risorse protette.
- Secret key JWT letta da variabili ambiente, non hardcoded, con consiglia uso di vault o secret manager per produzione.
- Scadenza token JWT configurabile via parametri (default 15-60 minuti).
- Endpoint protetti da dependency FastAPI che verifica token nell’header Authorization Bearer, decodifica e validazione.
- Gestione errori per token invalidi, scaduti o mancanti (401 Unauthorized).
- Reset password tramite token temporanei generati e salvati in DB, con scadenza breve (es. 1 ora).
- Mancata gestione revoca JWT o sessioni multiple per scope MVP.
- Nessuno throttling o limitazione tentativi, da valutare in evoluzioni per sicurezza.

## 9. Gestione errori
- Risposte HTTP con codici appropriati (400 Bad Request, 401 Unauthorized, 404 Not Found, 500 Internal Server Error).
- Messaggi di errore chiari e non esplicativi di dettagli sensibili (es. “Credenziali errate”, “Token non valido”).
- Gestione eccezioni di database (duplicate key, timeout) con conversione in risposte API significative.
- Validazione input tramite Pydantic con messaggi dettagliati per errori formali.
- Catch generale errori non gestiti per evitare crash e fornire messaggi generici 500.
- Nessun dettaglio di stacktrace in risposta per non esporre informazioni interne.
- Gestione errori concorrenza generica; miglioramenti sulla gestione dei token reset in concorrenza da valutare.

## 10. Strategia di test backend
- Test unitari per funzioni hashing password, generazione e validazione JWT, creazione e verifica token reset.
- Test unitari per modelli dati e validazione Pydantic input/output.
- Test di integrazione endpoint con DB reale/test (SQLite in memoria o test container PostgreSQL):
  - Registrazione con dati validi e invalidi (email malformata, password debole).
  - Login con credenziali corrette e sbagliate.
  - Accesso a endpoint protetti con token valido, scaduto, mancante.
  - Flusso reset password completo: richiesta token, reset con token valido, tentativi con token scaduto/errato.
- Test di sicurezza base: assicurare che password in chiaro non siano mai restituite da API.
- Test di carico minimi simulando richieste multiple per verificare stabilità DB pool e connessione.
- Test di error handling su casi di duplicati, token invalidi, eccezioni DB.
- Coverage test superiore al 80% per codice business-critical.
- Uso di framework pytest e FastAPI TestClient.

## 11. Rischi tecnici
- Flusso reset password base vulnerabile ad abusi senza workflow di validazione esterna (es. email).
- Assenza di throttling o limitazioni attacchi brute force su login e reset può compromettere sicurezza.
- Possibile esposizione del JWT secret se non gestito correttamente negli ambienti di produzione.
- Mancanza di revoca o refresh token espone a rischi in gestione sessioni multiple.
- Mancata gestione accurate del conflitti concorrenti nella invalidazione token reset.
- Assenza di logging e audit limita capacità di rilevamento accessi malevoli.
- Potenziale rischio di overflow DB se token reset non vengono puliti periodicamente (meccanismo di cleanup non previsto).
- Scalabilità non progettata per carichi elevati, da considerare versione successiva.
- Integrazione futura con sistemi email o MFA potrebbe richiedere refactoring parziale.

## 12. Struttura file proposta
```
/project-root
│
├── app/
│   ├── __init__.py
│   ├── main.py                 # entrypoint FastAPI
│   ├── config.py               # caricamento configurazioni da env
│   ├── database.py             # setup DB e sessioni ORM
│   ├── models.py               # ORM User model
│   ├── schemas.py              # Pydantic schemas request/response
│   ├── auth.py                 # funzioni hashing, JWT, reset token
│   ├── dependencies.py         # FastAPI dependencies per sicurezza
│   ├── routes/
│   │   ├── __init__.py
│   │   ├── auth.py             # /register, /login, reset password endpoints
│   │   └── protected.py        # endpoint protetto di esempio
│   └── migrations/             # Alembic folder per migrazioni
│
├── tests/
│   ├── __init__.py
│   ├── test_auth.py            # test unitari/integrativi auth, JWT, reset
│   ├── test_routes.py          # test funzionali endpoint
│   └── test_models.py          # test modelli e hashing
│
├── alembic.ini                 # configurazione Alembic
├── requirements.txt            # dipendenze progetto
├── .env                       # variabili ambiente (non versionato)
└── README.md                   # documentazione base
```

## 13. Piano di implementazione
1. Setup ambiente virtualenv, installazione dipendenze FastAPI, SQLAlchemy, Alembic, bcrypt, PyJWT.
2. Configurazione connessione PostgreSQL e setup ORM con database.py.
3. Definizione modello User con campi obbligatori e reset token.
4. Scrittura di migrazione Alembic per tabella utenti.
5. Implementazione funzioni hashing password e validazione JWT in auth.py.
6. Creazione endpoint POST /register con validazione dati e gestione errori duplicati.
7. Implementazione endpoint POST /login con verifica password e generazione JWT.
8. Configurazione dependency FastAPI per verifica token JWT e protezione endpoint.
9. Implementazione endpoint esempio GET /protected-resource protetto.
10. Sviluppo flussi base reset password: POST /reset-password-request per generare token; POST /reset-password per cambio password con token.
11. Scrittura test unitari e funzioniali per tutti gli endpoint e meccanismi critici.
12. Sviluppo documentazione API utilizzando OpenAPI generata automaticamente da FastAPI.
13. Revisione sicurezza base: verifica hashing, scadenze token, gestione errori.
14. Redazione documentazione/linee guida per gestione JWT secret in produzione (env var, best practices).
15. Refactoring finale per modularità e eventuale implementazione di miglioramenti indicati (es. throttling, logging).
16. Preparazione ambiente staging per test integrati prima del rilascio.