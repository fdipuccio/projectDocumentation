MODULE: BACKEND VERSION: 1
## 1. Obiettivo backend
Progettare e implementare un backend API user authentication basato su FastAPI e PostgreSQL, che consenta la registrazione e login via email e password, l’emissione e validazione di token JWT per sessioni protette, la protezione di endpoint riservati e una funzionalità base di reset password con token temporaneo. Il sistema deve garantire sicurezza minima con hashing password, gestione scadenza JWT, e fornire endpoint REST documentati con validazione dei dati.

## 2. Assunzioni tecniche
- Il sistema gestisce solo autenticazione tramite email e password.
- Le password sono salvate in forma hashed tramite algoritmo bcrypt o equivalente.
- I token JWT includono identificativo utente e scadenza (configurabile).
- Non si integrano sistemi esterni (es. invio email per reset).
- La gestione reset password si basa su token salvati temporaneamente in DB, senza workflow avanzati.
- Le verifiche sui dati (es. validità email, complessità password minima) sono applicate per ridurre errori comuni.
- La configurazione PostgreSQL utilizza ORM SQLAlchemy per interazione DB e migrazioni con Alembic (se previste).
- L’architettura è modulare per riuso e facile estensione.

## 3. Architettura backend
Architettura a 3 livelli:
1. **API layer**: FastAPI espone endpoint REST, con request/response validation tramite Pydantic.
2. **Business logic layer**: gestione core autenticazione, hashing password, JWT, logica reset password.
3. **Persistence layer**: accesso e persistenza dati utenti e token su PostgreSQL tramite SQLAlchemy ORM.
Il sistema includerà dipendenze o middleware FastAPI per validazione token JWT e protezione endpoint. L’API espone OpenAPI documentazione automatica.

## 4. Moduli e responsabilità
- **app.main**: avvio FastAPI, setup middleware e routing.
- **app.models**: definizione modelli SQLAlchemy (User, PasswordResetToken).
- **app.schemas**: Pydantic schemi per input/output API (UserCreate, Token, LoginData, PasswordReset).
- **app.database**: configurazione connessione DB e sessioni.
- **app.auth**: funzioni hashing password, generazione/validazione JWT, security dependencies FastAPI.
- **app.api.routes**: definizione endpoint:
  - registrazione
  - login
  - reset password (richiesta token e reset con token)
  - endpoint protetto esempio
- **app.core.config**: configurazioni centralizzate (DB URI, JWT secret, scadenze).
- **app.tests**: test unitari e funzionali principali.

## 5. API principali
| Metodo | Endpoint             | Descrizione                                              | Input                       | Output                          | Protezione JWT |
|--------|----------------------|----------------------------------------------------------|-----------------------------|-------------------------------|----------------|
| POST   | /register            | Registrazione utente nuova con email e password          | email, password             | messaggio di successo/errore   | No             |
| POST   | /login               | Autenticazione e generazione token JWT                    | email, password             | token JWT                      | No             |
| POST   | /password-reset      | Richiesta token reset password (salvato lato backend)     | email                       | messaggio conferma             | No             |
| POST   | /password-reset/confirm | Reset password: fornisce nuovo password con token reset  | token reset, nuova password | messaggio conferma/errore      | No             |
| GET    | /protected-resource  | Esempio endpoint protetto accessibile solo con JWT valido | -                           | dati risorsa esempio            | Si             |

## 6. Business logic
- **Registrazione**: validazione email formale, validità password minima, controllo univocità email, hashing password, salvataggio utente.
- **Login**: verifica email presente, confronto password hashed con bcrypt, generazione token JWT contenente id utente e scadenza.
- **JWT verification**: decoding token con secret, verifica integrità e scadenza, estrazione utente.
- **Reset password**:
  - Creazione token reset (UUID o stringa random), salvato nel DB con scadenza breve (es. 15 min).
  - Validazione token in endpoint di conferma reset.
  - Aggiornamento password con nuova password hashed.
- **Protezione endpoint**: dipendenza FastAPI che estrae, decodifica e valida token JWT per accesso protetto.

## 7. Persistenza e integrazioni
- Utilizzo PostgreSQL come DB relazionale.
- Modello User con campi: id UUID, email (unique), hashed_password, data creazione, eventualmente campo per reset_token e scadenza correlata (oppure tabella separata PasswordResetToken con FK a User).
- Connessione gestionata tramite SQLAlchemy ORM + sessioni SQLAlchemy.
- Migrazioni DB gestite con Alembic (baseline minima per schema user).
- Niente integrazione esterna per ora (es. invio email).
- Connessione DB affidabile con pooling e riconnessione automatica.

## 8. Autenticazione e autorizzazione
- Autenticazione via email e password.
- Password hashing sicuro tramite bcrypt.
- JWT firmato con secret configurabile e algoritmo HS256.
- JWT payload contiene almeno user_id e exp (expiration timestamp).
- Validazione JWT implementata come dependency FastAPI, espone user corrente solo se token valido.
- Endpoints protetti richiedono token JWT valido in header Authorization Bearer.
- Reset password gestito tramite token temporaneo generato e salvato lato backend, senza workflow esterno.

## 9. Gestione errori
- Risposta standardizzata con codici HTTP appropriati:
  - 400 Bad Request per dati invalidi.
  - 401 Unauthorized per credenziali errate o token mancanti/invalidi.
  - 404 Not Found per utenti non esistenti o token reset invalidi.
  - 500 Internal Server Error per errori imprevisti.
- Messaggi errore chiari e non ridondanti.
- Validazione Pydantic per dati di input.
- Gestione eccezioni per casi comuni come duplicazione email, token scaduto o non trovato.
- Logging minimo delle eccezioni.

## 10. Strategia di test backend
- Test unitari per:
  - Funzioni hashing password, verifica passwd.
  - Generazione e verifica JWT.
  - Creazione token reset e validità.
- Test funzionali API tramite TestClient FastAPI:
  - Registrazione utente corretta e con dati errati.
  - Login con credenziali valide e non valide.
  - Accesso endpoint protetto con JWT valido e senza.
  - Flusso reset password: richiesta token e reset con token valido e non.
- Test di sicurezza base: verifica non esposizione password in chiaro, scadenza token effettiva.
- Test database in memoria o test PostgreSQL dedicato.

## 11. Rischi tecnici
- Ambiguità nel flusso reset password e potenziale vulnerabilità derivata da workflow semplificato.
- Gestione segreta JWT e sicurezza: secret key deve essere gestito in modo sicuro (env vars).
- Possibili attacchi brute force su login o reset password.
- Gestione token revoca o refresh non prevista.
- Assenza di restrizioni avanzate su forza password.
- Scalabilità non prevista come requisito, ma può limitare performance su carichi elevati.
- Mancata integrazione con sistemi email limita usabilità reale del reset.

## 12. Struttura file proposta
```
app/
├── main.py                  # entrypoint FastAPI, avvio app, inclusione router
├── models.py                # modelli ORM SQLAlchemy User, PasswordResetToken
├── schemas.py               # schemi Pydantic input/output
├── database.py              # creazione sessioni DB, connessione
├── auth.py                  # funzioni auth: hashing, JWT, dependencies
├── api/
│   └── routes.py            # definizione endpoints register, login, reset, protetto
├── core/
│   └── config.py            # configurazioni centralizzate (secret, db url, scadenze)
└── tests/
    ├── test_auth.py         # test unitari/auth
    ├── test_api.py          # test funzionali endpoint
    └── __init__.py
alembic/                      # migrazioni DB Alembic
```

## 13. Piano di implementazione
1. Setup ambiente FastAPI, virtualenv e progetto base con struttura file.
2. Configurare connessione PostgreSQL e creare modello User con SQLAlchemy.
3. Impostare hashing password con bcrypt e prime funzioni utilitarie auth.
4. Implementare endpoint registrazione con validazione dati e salvataggio utente.
5. Implementare endpoint login con verifica password e generazione token JWT.
6. Implementare middleware/dependency FastAPI per validazione token JWT su chiamate protette.
7. Creare endpoint protetto esempio per accesso con token valido.
8. Progettare e implementare flusso base reset password: generazione token reset + applicazione reset con token.
9. Scrivere test unitari e funzionali per tutti i casi principali.
10. Realizzare documentazione API tramite FastAPI/OpenAPI.
11. Verificare sicurezza minima: hashing, validazione token, scadenze token.
12. Revisione codice e test completi pre-release.