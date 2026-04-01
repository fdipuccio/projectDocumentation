MODULE: BACKEND VERSION: 1
## 1. Obiettivo backend
Progettare e realizzare un sistema backend API RESTful per la gestione dell'autenticazione utenti, basato su FastAPI e PostgreSQL, che consenta la registrazione, il login tramite email e password, la generazione e validazione di token JWT per accesso protetto agli endpoint, e una implementazione base del meccanismo di reset password tramite token. L’obiettivo è fornire un backend sicuro, semplice, facilmente estendibile e conforme agli standard di sicurezza di base per ambienti enterprise non critici.

## 2. Assunzioni tecniche
- Lato sicurezza, si applicheranno standard consolidati per hashing password (bcrypt o equivalente).
- Per reset password si utilizzerà un flusso semplificato basato su token memorizzati e validati in database senza obbligo di implementare il delivery email.
- JWT verrà utilizzato per autenticare utenti tramite header Authorization; non si gestisce scadenza avanzata né refresh token.
- L’invio email per reset password è fuori dallo scope, previsto per essere integrato da altri team.
- Non saranno implementate logiche di ruoli, MFA, social login o policy complesse sul formato password.
- Il sistema rimarrà un API server senza UI/Frontend integrato.
- La configurazione e segreti (chiave JWT, DB credentials) saranno gestiti con variabili ambiente.

## 3. Architettura backend
La soluzione si basa su un’architettura monolitica modulare FastAPI con separazione chiara fra:
- Layer di persistenza: ORM SQLAlchemy con modello utenti su PostgreSQL.
- Layer di servizi: Logica di business per registrazione, autenticazione, reset password.
- Layer di API: Endpoints REST responsabili di input validation, invocazione servizi e gestione risposte.
- Middleware/Dependency injection FastAPI per autenticazione JWT e protezione endpoint.
L’architettura favorisce la riusabilità del codice tramite moduli Python, separati per controller, servizi, modelli e utilità.

## 4. Moduli e responsabilità
- **models.py**: Definizione modello User ORM con campi email, password_hash, reset_token, timestamp.
- **schemas.py**: Pydantic per validazione input/output API.
- **database.py**: Configurazione engine e sessione PostgreSQL.
- **crud.py**: Funzioni CRUD per la gestione utenti e token reset.
- **auth.py**: Logica hashing password, generazione e verifica token JWT.
- **dependencies.py**: Dependency injection FastAPI per connessione DB, autenticazione JWT.
- **api/routes/auth.py**: Endpoint registrazione, login e reset password.
- **api/routes/protected.py**: Endpoint di esempio protetti JWT.
- **main.py**: Configurazione FastAPI, montaggio rotte e middleware.
- **tests/**: Test unitari e integrativi per endpoint, logiche sicurezza password e token.

## 5. API principali
- `POST /auth/register`: Registrazione nuovo utente (email, password) con validazione base, salvataggio sicuro password.
- `POST /auth/login`: Login con email e password, restituisce token JWT in risposta JSON.
- `POST /auth/reset/request`: Richiesta reset password, genera token reset associato utente (token a scopo demo, non invio email).
- `POST /auth/reset/confirm`: Conferma reset password tramite token reset e nuova password.
- Endpoint protetti di esempio (es. `GET /protected/profile`) accessibili solo con header Authorization contenente token JWT valido.

## 6. Business logic
- Registrazione verifica email non duplicata, cifra password con bcrypt o algoritmo equivalente.
- Login valida credenziali e genera JWT con payload contenente user_id ed email.
- Reset password gestisce creazione e memorizzazione token univoco temporaneo (es. UUID, timestamp) associato all’utente.
- La conferma reset valida token, verifica scadenza (opzionale base), aggiorna password cifrata.
- Protezione endpoints tramite verifica token JWT e gestione errori di autenticazione.

## 7. Persistenza e integrazioni
- PostgreSQL come database relazionale per persistenza utenti e token reset password.
- SQLAlchemy ORM per gestione modelli/dati, sessioni e transazioni.
- Nessuna integrazione esterna diretta prevista, l’invio email sarà delegato ad altra componente.
- La struttura DB userà una tabella `users` con:
  - id (PK)
  - email (unique)
  - password_hash
  - reset_token (nullable)
  - reset_token_expiry (nullable datetime)
  - created_at, updated_at timestamp

## 8. Autenticazione e autorizzazione
- Autenticazione stateless basata su token JWT firmati con chiave segreta configurabile tramite variabili ambiente.
- JWT con payload minimale per identificazione utente, senza refresh token o ruoli.
- Middleware/dipendenza FastAPI che estrae token da header Authorization Bearer, decodifica, verifica validità e carica utente.
- Endpoint protetti restituiscono 401 in assenza o invalidità token.
- Password utente cifrate con algoritmo sicuro (bcrypt) con salatura.

## 9. Gestione errori
- Utilizzo di risposte HTTP con codici standard:
  - 400 Bad Request per input invalidi o dati mancanti.
  - 401 Unauthorized quando token JWT mancante o invalido.
  - 404 Not Found per token reset scaduti o utenti non trovati.
  - 500 Internal Server Error per errori non gestiti.
- Messaggi di errore chiari in formato JSON.
- Logging degli errori per debugging eventuale.

## 10. Strategia di test backend
- Test unitari per funzioni di hashing, generazione e verifica token JWT, CRUD utenti.
- Test integrativi per endpoint REST: registrazione, login, richiesta reset, conferma reset, endpoint protetti.
- Test copertura flussi nominali e casi limite (es. email duplicata, token reset non valido).
- Uso di client FastAPI TestClient e database in-memory / test PostgreSQL separato.
- Automatizzazione con framework pytest per esecuzione rapida e ripetibile.

## 11. Rischi tecnici
- Ambiguità flusso reset password potrebbe richiedere chiarimenti per integrazione email futura.
- Mancata definizione scadenza token JWT o reset token potrebbe generare rischi di sicurezza se non parametrizzati.
- Assenza di gestione avanzata sicurezza (es. rate limiting) espone a vulnerabilità di sicurezza brute force.
- Nessuna policy password obbligatoria potrebbe portare a password deboli.
- Eventuali modifiche a stack, volumi utenti o richieste flow di autenticazione future potrebbero richiedere architettura più scalabile.

## 12. Struttura file proposta
```
/app
  /api
    /routes
      auth.py
      protected.py
  /core
    config.py                # gestione configurazioni e variabili ambiente
    security.py              # logica hashing, JWT e reset token
    dependencies.py          # dependency injection FastAPI
  /crud
    user.py                  # funzioni CRUD utenti e reset token
  /db
    base.py                  # Base ORM e sessione DB
    models.py                # modelli ORM (User)
  /schemas
    user.py                  # Pydantic schemas input/output
  /tests
    test_auth.py
    test_reset.py
    test_protected.py
  main.py                    # entrypoint FastAPI
```

## 13. Piano di implementazione
1. Definizione schema database utenti e modello ORM su PostgreSQL.
2. Setup ambiente FastAPI e configurazione connessione DB.
3. Implementazione hashing password e funzioni CRUD base.
4. Endpoint registrazione utente con validazione base.
5. Endpoint login con verifica credenziali e generazione token JWT.
6. Implementazione middleware/dipendenza per protezione endpoint con JWT.
7. Endpoint reset password: richiesta token reset e conferma con nuova password.
8. Scrittura di test automatici unitari e integrativi per coprire i flussi principali.
9. Scrittura documentazione API tramite OpenAPI/Swagger auto-generata.
10. Revisione sicurezza base: gestione segreti, hashing, errore handling.
11. Debug, test di integrazione finale e refactoring per code reuse e manutenzione.