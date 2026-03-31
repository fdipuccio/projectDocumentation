MODULE: BACKEND VERSION: 1
## 1. Obiettivo backend
Progettare e sviluppare un backend API RESTful per gestire l’autenticazione utenti in un contesto enterprise, basato su FastAPI e PostgreSQL, che consenta registrazione, login via email e password, generazione/verifica token JWT per protezione sessioni, e una procedura base di reset della password tramite token temporanei memorizzati e validati internamente. La soluzione deve rispettare requisiti di sicurezza base, garantire una buona manutenibilità e essere conforme alle specifiche OpenAPI generata automaticamente.

## 2. Assunzioni tecniche
- Utilizzo del framework Python FastAPI per la creazione dell’API backend con standard REST.
- Persistenza dati utenti su PostgreSQL tramite ORM SQLAlchemy.
- Email utente univoca, usata come chiave primaria logica per identificazione.
- Password salvate solo in forma hashed con algoritmo bcrypt (o argon2 come alternativa).
- JWT generati e verificati con algoritmo HS256 e segreto configurabile, includendo esplicitamente la gestione della scadenza per sicurezza.
- Flusso reset password base: utente richiede token reset, token temporaneo valido per un tempo limitato, reset password usando token stesso; non è implementato invio automatico di email.
- I limiti di complessità password sono definiti: minimo 8 caratteri, con almeno un numero e una lettera (linea guida esplicita da documentare).
- Non sono previste gestione di ruoli, permessi, MFA, OAuth o altri sistemi di autenticazione social.
- Endpoint protetti tramite dependency FastAPI che verifica e valida il JWT.
- Documentazione API generata automaticamente con OpenAPI/Swagger.
- Testing automatico per flussi principali, inclusi casi negativi, usando SQLite in memory con mocking.

## 3. Architettura backend
Architettura modulare con separazione chiara di:
- Layer di presentazione (API routing FastAPI)
- Business logic centrale (application services per gestione flussi registro/login/reset)
- Persistenza dati orientata a oggetti tramite ORM SQLAlchemy (modelli e repository)
- Modulo sicurezza per hashing password e gestione token JWT
- Dependency injection per componenti chiave (DB session, configurazione, autenticazione)
- Gestione degli errori centralizzata con risposte HTTP significative
- Testing isolato dei moduli più critici per garantire robustezza
L’architettura tiene conto delle best practice per mantenere codice riusabile, testabile ed estendibile.

## 4. Moduli e responsabilità
- **models**: definizione entità utenti, campi email, password hash, reset token e scadenza
- **schemas**: Pydantic models per request/response validation (register, login, reset password)
- **crud**: funzioni per operazioni su DB: creazione utente, ricerca, aggiornamento password e token reset
- **security**: hashing password (bcrypt), gestione JWT (generazione, verifica con scadenza)
- **api/auth**: endpoint per registrazione (/register), login (/login), reset password (/reset-request, /reset-confirm)
- **api/protected**: esempi endpoint protetti (/protected-demo) che richiedono autenticazione JWT
- **dependencies**: dependency FastAPI per validazione token JWT sui percorsi protetti
- **config**: gestione configurazioni (chiave segreta JWT, algoritmo, parametri token, limiti password)
- **tests**: test unitari e di integrazione per tutti i flussi principali con mocking DB

## 5. API principali
- `POST /register`: registra utente con email e password; valida unicità email, complessità password, risponde 201 o 422
- `POST /login`: accetta email e password, verifica credenziali e restituisce token JWT (in risposta JSON)
- `POST /reset-request`: richiede generazione token di reset password valido per tempo limitato; token salvato DB per utente
- `POST /reset-confirm`: accetta token reset e nuova password, valida token e scadenza, aggiorna password cifrata e cancella token
- `GET /protected-demo`: endpoint di esempio accessibile solo con token JWT valido; risponde 401 se token mancante o invalido

## 6. Business logic
- Registrazione valida unicità email, validazione formale email e complessità password minima (>=8 char, lettere e numeri)
- Password cifrata usando bcrypt con salatura automatica prima di memorizzazione
- Login verifica password tramite confronto hash bcrypt, genera JWT con payload user_id, email e scadenza configurabile (es. 15-30 minuti)
- Reset password crea token UUID random, memorizza con scadenza (es. 1 ora) in record utente, permette reset solo con token valido e non scaduto
- Token JWT viene validato in ogni richiesta protetta tramite dependency che solleva 401 su errori firma o scadenza
- Rigetto chiaro e consistente di richieste non autorizzate o con dati errati, mantenendo sicurezza ed esperienza d’uso

## 7. Persistenza e integrazioni
- Database PostgreSQL con tabella utenti:
  - id (UUID o serial PK)
  - email (stringa unica, indice)
  - password_hash (stringa)
  - reset_token (stringa nullable)
  - reset_token_expiry (timestamp nullable)
- ORM SQLAlchemy con session management per operazioni atomiche e rollback in caso di errori
- Nessuna integrazione esterna prevista: reset password funziona interamente tramite token interni senza invii email

## 8. Autenticazione e autorizzazione
- Autenticazione tramite JWT firmati con HS256 e segreto definito in ambiente/config
- Payload JWT minimale contenente user_id e data scadenza esplicita (`exp`)
- Validazione token: verifica firma, integrità e scadenza. In caso di token mancante/errato/expirato, risposta HTTP 401
- Dependency FastAPI dedicata a validazione e parsing token, fornisce utente corrente agli endpoint protetti
- Nessun sistema di autorizzazione o ruoli, accesso garantito a tutti gli utenti autenticati

## 9. Gestione errori
- Risposte HTTP standard coerenti con tipo errore:
  - 400 Bad Request per dati invalidi o sintassi errata
  - 401 Unauthorized per problemi di autenticazione/token
  - 404 Not Found per risorse utenti non trovate (es. token reset non valido)
  - 422 Unprocessable Entity per validazioni fallite (es. email duplicata, password debole)
  - 500 Internal Server Error per errori imprevisti (minimizzati e loggati internamente)
- Body delle risposte errore con messaggi chiari e codificati, utili per client
- Endpoint protetti rispondono con messaggi e codice 401 standard se token JWT è mancante, scaduto, o corrotto; documentato chiaramente nella API

## 10. Strategia di test backend
- Test unitari per business logic (hashing password, validazione JWT, generazione token reset)
- Test CRUD con mocking DB SQLite in memoria, testando create/read/update/delete utenti e token reset
- Test di integrazione end-to-end per:
  - registrazione utente (successo, email duplicata, password debole)
  - login (successo, fallo con credenziali errate)
  - accesso endpoint protetti con token valido e invalidi (mancanti, scaduti, alterati)
  - reset password: richiesta token, uso corretto token, uso token scaduto/errato, cancellazione del token dopo reset
- Verifica della generazione della documentazione OpenAPI e corretto funzionamento Swagger UI generato
- Test di sicurezza di base per assicurarsi che password non sia mai memorizzata in chiaro
- Include test con carichi simili per verificare stabilità (base, vista come prerequisito futuro scalabilità)

## 11. Rischi tecnici
- Flusso reset password base senza invio email espone rischio basso di sovrascrittura password da parte di soggetti malintenzionati con accesso al reset token che però è temporaneo e valido per breve periodo
- Mancanza di politiche di rate limiting e anti-brute force espone a rischio attacchi su login e reset password (da considerare implementazioni future)
- Assenza di verifica email può creare account con email non validi o falsi
- Mancanza di rotazione o refresh token JWT può comportare esposizione più lunga periodi di token compromessi
- Logging e auditing operazioni puntano a essere implementati per migliore tracciamento e investigazione
- Controllo simultaneo o multipla richiesta reset token deve essere gestito per evitare sovrascritture errate che invaliderebbero token già emessi

## 12. Struttura file proposta
```
app/
 ├── main.py                     # FastAPI app entrypoint
 ├── api/
 │    ├── auth.py                # router con /register, /login, /reset-request, /reset-confirm
 │    └── protected.py           # router per endpoint protetti demo (/protected-demo)
 ├── models/
 │    └── user.py                # SQLAlchemy modello User
 ├── schemas/
 │    └── user.py                # Pydantic schemas (UserCreate, UserLogin, ResetRequest, ResetConfirm)
 ├── crud/
 │    └── user.py                # funzioni CRUD utenti e gestione reset token
 ├── security/
 │    ├── hashing.py             # funzioni hash e verifica password
 │    └── jwt_handler.py         # generazione/validazione JWT
 ├── dependencies.py             # dipendenze FastAPI (db session, get_current_user)
 ├── config.py                  # gestione configurazioni (env, segreti, parametri)
 ├── tests/
 │    ├── test_auth.py           # test registrazione, login, reset password
 │    └── test_protected.py      # test endpoint protetti e token
 └── utils/                      # eventuali helper (es. validatori password)
```

## 13. Piano di implementazione
1. Definire e implementare schema utenti in SQLAlchemy con campi e vincoli (email unica, campi reset token)
2. Setup ambiente FastAPI e configurazione base (config.py, DB connection, main.py)
3. Implementare hashing password sicuro (bcrypt) e test unitari relativi
4. Implementare gestione token JWT (generazione con `exp`, verifica con scadenza) e relativi test
5. Sviluppare endpoint `/register` con validazione email e password, gestione errori specifici
6. Sviluppare endpoint `/login` con verifica credenziali e restituzione JWT
7. Creare dependency FastAPI per protezione endpoint basata su JWT
8. Creare endpoint protetto di esempio `/protected-demo` per validazioni accessi
9. Implementare flusso reset password con endpoint `/reset-request` (creazione token reset) e `/reset-confirm` (uso token e nuovo hash password), con gestione scadenza token
10. Scrivere test integrazione completi su tutti i flussi fondamentali (registrazione, login, accesso protetto, reset password)
11. Verificare e migliorare messaggi di errore HTTP coerenti alle situazioni
12. Generare e validare documentazione OpenAPI/Swagger garantendo completezza e coerenza degli endpoint
13. Revisione finale e documentazione sulle limitazioni di sicurezza note, linee guida per futuri miglioramenti (rate limiting, logging, verifica email)
14. Consegna e supporto per integrazione nell’ecosistema complessivo aziendale

Questa proposta rispetta appieno requisiti, feedback QA, stack tecnologico e vincoli specifici, garantendo un backend robusto, sicuro e modulare per l’autenticazione utenti enterprise con FastAPI e PostgreSQL.