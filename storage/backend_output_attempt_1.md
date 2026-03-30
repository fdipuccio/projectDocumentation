MODULE: BACKEND VERSION: 1
## 1. Obiettivo backend
Progettare e implementare un backend API RESTful per la gestione dell’autenticazione utenti basata su email e password, garantendo le funzionalità di registrazione, login con rilascio di token JWT, protezione di endpoint con autenticazione JWT, e una funzionalità base di reset password tramite token. Il sistema deve utilizzare FastAPI, PostgreSQL, e JWT, assicurando sicurezza basilare nell’hashing password e validazione token.

## 2. Assunzioni tecniche
- L’email è un identificatore univoco per ogni utente.
- La password sarà hashata in modo sicuro (bcrypt) e non salvata in chiaro.
- JWT sarà usato per session management, con claims standard, durata da definire internamente (es. 15-60 minuti).
- Il reset password è a livello base: generazione token univoco associato a utente, senza invio email integrato.
- Ogni endpoint REST rispetta i principi di FastAPI e restituisce risposte JSON coerenti.
- L’ambiente di sicurezza (HTTPS, segretezza chiavi JWT, revoca token) è gestito a livello infrastrutturale e non fa parte del backend.
- Non sono previsti sistemi complessi di autorizzazione (ruoli o permessi avanzati) né MFA.
- Validazioni input minime: email valida, password con lunghezza minima (es. 8 caratteri).
- Errori client definiti e coerenti con standard HTTP.

## 3. Architettura backend
Architettura modulare monolitica, suddivisa in layer:
- API Layer: gestione delle richieste HTTP tramite FastAPI, definizione degli endpoint.
- Application Layer: implementazione business logic per autenticazione, registrazione, reset password.
- Data Layer: interazione con database PostgreSQL tramite ORM (ad esempio SQLAlchemy o equivalent).
- Security Layer: gestione hashing password, generazione e validazione JWT, token reset password.
- Middleware e dipendenze FastAPI per protezione endpoint e parsing JWT.

Il backend esporrà esclusivamente API RESTful JSON. Utilizzo di migrator DB (es. Alembic) per gestione schema.

## 4. Moduli e responsabilità
- **models.py**: definizione modello utente e modello token reset password per DB.
- **schemas.py**: definizione dei modelli Pydantic per validazione e serializzazione input/output.
- **database.py**: configurazione connessione PostgreSQL e sessioni ORM.
- **auth.py**: funzioni hashing password, generazione/validazione JWT, generazione token reset.
- **crud.py**: funzioni per operazioni CRUD su utenti e token.
- **main.py**: configurazione FastAPI, definizione router API, inclusione middleware.
- **routers/auth_routes.py**: endpoint REST per registrazione, login, reset password.
- **dependencies.py**: dipendenza FastAPI per verifica autenticazione JWT e protezione endpoint.
- **tests/**: test integrati per validazione funzionalità e sicurezza base.

## 5. API principali
- **POST /register**
  - Input: email, password
  - Azione: crea utente, hash password, salva DB
  - Output: conferma registrazione o errore duplicato/invalid

- **POST /login**
  - Input: email, password
  - Azione: verifica credenziali, emette JWT con scadenza
  - Output: token JWT o errore autenticazione

- **GET /protected**
  - Proteggo con JWT
  - Output: semplice conferma accesso autenticato

- **POST /password-reset/request**
  - Input: email
  - Azione: genera token reset password associato utente (validità limitata)
  - Output: token reset (nella risposta o log, in assenza invio email)

- **POST /password-reset/confirm**
  - Input: token reset, nuova password
  - Azione: verifica token reset valido, aggiorna password hashata, invalida token
  - Output: conferma reset o errore token/password

Tutte le risposte saranno JSON, con codici HTTP appropriati (200, 201, 400, 401, 404).

## 6. Business logic
- Durante la registrazione si valida email e password (lunghezza minima).
- La password viene hashata con bcrypt prima di memorizzarla.
- Login verifica password hashed e, in caso positivo, crea un JWT firmato con claims standard e durata configurabile.
- I token JWT vengono verificati su endpoint protetti tramite dipendenza FastAPI.
- Il reset password prevede la creazione di un token unico (es. UUID o token random crittografico), salvato su DB con riferimenti a utente e scadenza.
- Solo con token reset valido si permette modifica password.
- Token reset viene invalidato dopo uso o dopo scadenza.
- Nessun invio email: token reset può essere restituito nella risposta o gestito in altro modo da integrazione futura.

## 7. Persistenza e integrazioni
- Database PostgreSQL configurato con connessione gestita tramite ORM (SQLAlchemy raccomandato).
- Modello utente contiene almeno id, email, password_hash, data_creazione, data_ultimo_accesso.
- Modello token reset password contiene token univoco, utente_id, data_creazione, data_scadenza, stato (valido/invalidato).
- Migrazioni schema con Alembic o altro tool standard per garantire versioning DB.
- Nessuna integrazione esterna prevista per MVP (es. non si integra con email/sms provider).

## 8. Autenticazione e autorizzazione
- Autenticazione via JWT standard firmato con chiave segreta configurata a livello ambiente.
- Token contiene claims standard: sub (user_id), iat, exp, iss (opzionale).
- Endpoint protetti richiamano dipendenza FastAPI che estrae il token dall’header Authorization Bearer, verifica firma e scadenza.
- Se token non valido/assente, l’accesso è negato con errore 401.
- Non è prevista autorizzazione granulari o ruoli; accesso è “autenticato” vs “non autenticato”.
- Password hashata con bcrypt con cost parametri per bilanciare sicurezza e performance.

## 9. Gestione errori
- Input invalidi restituiscono 422 Unprocessable Entity (FastAPI gestisce automaticamente).
- Errore autenticazione (login fallito, token JWT non valido) restituisce 401 Unauthorized con messaggio generico che non riveli informazioni sensibili.
- Tentativi di registrazione con email già esistente restituiscono 400 Bad Request con messaggio esplicativo.
- Token reset password scaduti o invalidi restituiscono 400 o 401 con messaggi chiari.
- Errori interni server 500 gestiti con risposta generica, log lato backend per troubleshooting.
- Tutte le risposte di errore in formato JSON con chiave "detail" per coerenza standard FastAPI.

## 10. Strategia di test backend
- Test unitari su funzioni hashing, generazione JWT e token reset.
- Test integrati API endpoint principali per:
  - Registrazione con input corretti e duplicati.
  - Login con credenziali valide e non valide.
  - Accesso endpoint protetti con e senza token.
  - Flusso reset password completo: richiesta token, modifica password con token valido/invalidi.
- Test sicurezza base: conferma che password non è mai restituita, token JWT scaduto rifiutato.
- Utilizzo di client di test FastAPI (TestClient) e database di test isolato (es. sqlite in-memory o schema postgres dedicato).
- Coverage critico su regole di business e sicurezza.

## 11. Rischi tecnici
- Limiti di sicurezza derivanti da gestione token reset internamente senza email; possibile uso improprio o divulgazione token.
- Mancanza di gestione revoca token JWT: gli accessi rimangono validi fino a scadenza.
- Possibile ambiguità nella durata e claims specifici JWT, da definire o parametrizzare.
- Assenza di logging dettagliato e auditing può limitare il troubleshooting in produzione.
- Gestione errori deve essere implementata rigorosamente per evitare leak di informazioni.
- Dipendenza da configurazione ambientale per chiavi JWT, DB connection; errata configurazione impatta sicurezza e stabilità.

## 12. Struttura file proposta
```
/app
  ├── main.py                   # entrypoint FastAPI, setup api e app
  ├── database.py               # setup connessione DB e sessioni ORM
  ├── models.py                 # definizioni ORM utenti e token reset
  ├── schemas.py                # modelli Pydantic per request/response
  ├── auth.py                   # hashing password, JWT, token reset
  ├── crud.py                   # funzioni CRUD su utenti e token
  ├── routers
  │    └── auth_routes.py       # endpoint auth, login, register, reset password
  ├── dependencies.py           # dipendenze FastAPI per auth JWT
  ├── alembic/                  # cartella migrazioni schema DB (se Alembic)
  └── tests/
       ├── test_auth.py         # test funzionali e di sicurezza
       └── conftest.py          # setup test database/test client
```

## 13. Piano di implementazione
1. Setup ambiente FastAPI e configurazione connessione PostgreSQL (database.py + main.py).
2. Definizione modello dati utente e token reset con migrazioni.
3. Sviluppo moduli hashing password e JWT token generation/validation.
4. Implementazione endpoint registrazione con validazione input e hashing.
5. Implementazione endpoint login con generazione token JWT.
6. Configurazione dipendenza FastAPI per protezione endpoint (JWT verification).
7. Implementazione endpoint protetto di test accesso autenticato.
8. Implementazione reset password base:
    - Endpoint generazione token reset.
    - Endpoint modifica password con token reset.
9. Scrittura e esecuzione test unitari e integrati per tutte le componenti e flussi.
10. Verifica e aggiustamenti gestione errori e messaggi.
11. Generazione documentazione API OpenAPI automatica con FastAPI.
12. Revisione codice e deploy in ambiente test.

Tale piano garantisce copertura funzionale MVP entro i vincoli indicati, con flussi ben definiti e codice manutenibile.