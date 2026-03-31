MODULE: BACKEND VERSION: 1

## 1. Obiettivo backend
Sviluppare un backend API REST per la gestione dell’autenticazione utenti mediante registrazione, login con email e password, generazione e validazione di token JWT per l’accesso a risorse protette, e funzionalità di reset password base. Il sistema dovrà essere realizzato con FastAPI, con persistenza dati su PostgreSQL, garantendo sicurezza minima necessaria per la protezione delle credenziali e l'accesso autorizzato.

## 2. Assunzioni tecniche
- L’identificativo univoco utente è l’email.
- La password sarà salvata in DB in forma hashata (es. bcrypt).
- I token JWT contengono payload minimo con identificativo utente e scadenza e sono firmati con chiave segreta.
- Non è richiesto supporto per MFA o autenticazione social.
- Le email per reset password sono gestite esternamente.
- Le API sono documentate automaticamente tramite FastAPI/OpenAPI.
- Il progetto utilizza Python 3.8+ con FastAPI e librerie standard per JWT (es. PyJWT).
- Il database PostgreSQL gestisce le informazioni utenti e i dati per reset password (token o flag).
- La sicurezza si limita a standard base (hash password, firma JWT, verifica token).

## 3. Architettura backend
Il backend è un’applicazione FastAPI stateless che espone API REST protette tramite autenticazione JWT.  
La persistenza degli utenti è affidata a PostgreSQL tramite ORM (es. SQLAlchemy).  
Il flusso base prevede:
- Registrazione: validazione input, hashing password, salvataggio utente.
- Login: verifica credenziali, emissione token JWT.
- Middleware o dipendenza FastAPI per validazione token JWT su endpoint protetti.
- Endpoint di reset password divisi in richiesta e aggiornamento password.
L’architettura è modulare con separazione netta tra modelli dati, logica business, API e sicurezza.

## 4. Moduli e responsabilità
- **models.py**: definizione ORM per utenti e tabelle reset password.
- **database.py**: configurazione e sessione PostgreSQL.
- **schemas.py**: definizioni Pydantic per request/response (es. login, registrazione).
- **auth.py**: gestione hashing password, generazione e validazione token JWT.
- **dependencies.py**: dipendenze FastAPI per protezione endpoint (es. verifica token).
- **routers/users.py**: endpoint registrazione, login, reset password.
- **routers/protected.py**: endpoint di esempio protetti da JWT.
- **main.py**: setup applicazione FastAPI, inclusione router, middleware.
- **tests/**: test unitari e integrazione per principali flussi.
  
## 5. API principali
- **POST /register**: registra nuovo utente (email, password).
- **POST /login**: verifica credenziali, restituisce token JWT.
- **POST /reset-password/request**: avvia processo reset password (flag/token di reset).
- **POST /reset-password/confirm**: aggiorna password con nuova password fornita.
- **GET /protected**: endpoint protetto, accessibile solo con JWT valido.

Tutte le richieste input sono validate tramite Pydantic. La protezione JWT è obbligatoria per gli endpoint /protected e le risorse future protette.

## 6. Business logic
- **Registrazione**: verifica unicità email, valida formato e criteri base, hash password con bcrypt, salva nuovo utente.
- **Login**: verifica esistenza utente, confronto hash password, genera token JWT con payload (user_id, exp).
- **JWT**: firma con algoritmo sicuro (HS256) e chiave segreta mantenuta in configurazione.
- **Reset password**: 
  - `request`: salva flag o token (es. UUID temporaneo) nel DB associato all’utente per abilitare reset.
  - `confirm`: verifica flag/token valido, invalida flag, aggiorna password hashata.

## 7. Persistenza e integrazioni
- PostgreSQL come unico datastore.
- Tabelle:
  - **users**: id (PK), email (UNIQUE), password_hash, data_creazione, etc.
  - **password_reset_tokens** (opzionale): id, user_id (FK), token string, data_creazione, valido (flag).
- Nessuna integrazione esterna prevista, inclusa gestione email per reset password.

## 8. Autenticazione e autorizzazione
- Gestione autenticazione tramite token JWT firmati.
- Token JWT generati in login e validati in ogni richiesta ad endpoint protetti.
- Middleware o dipendenza FastAPI valida token estratto da header `Authorization: Bearer <token>`.
- In caso di token mancante, scaduto o errato viene restituito HTTP 401 Unauthorized.
- Non sono previsti ruoli o livelli di autorizzazione, solo verifica autenticazione.

## 9. Gestione errori
- Errore 400 Bad Request per dati input non validi (es. email malformata, password mancante).
- Errore 401 Unauthorized per tentativi di accesso utenze non autenticate o token JWT non validi.
- Errore 409 Conflict per email duplicate in fase di registrazione.
- Errore 404 Not Found per utente non esistente durante login o reset password.
- Risposte di errore chiare con messaggi utili ma non dettagli sensibili.
- Logging errori critici lato backend per monitoraggio.

## 10. Strategia di test backend
- Test unitari per:
  - Funzioni hashing password e verifica.
  - Generazione e verifica token JWT, gestione scadenza e payload.
  - Modelli dati e validazioni input.
- Test di integrazione per:
  - Registrazione utente, compresa gestione email esistente.
  - Login corretto e errato, emissione token.
  - Accesso endpoint protetti con token valido, token scaduto o assente.
  - Reset password: richiesta e conferma con validazioni.
- Utilizzo di database di test isolato.
- Automazione test tramite pytest o simile framework.

## 11. Rischi tecnici
- Mancanza di informazioni su politica password e sicurezza token potrebbe portare a implementazioni insufficienti.
- Assenza di gestione token revoca o blacklist può esporre a rischi in caso di compromissione.
- Reset password base senza invio email o validazioni può ridurre usabilità e sicurezza.
- Possibile conflitto email utenti non gestito correttamente.
- In assenza di dettagli su versioni librerie, possibili incompatibilità future.

## 12. Struttura file proposta
```
/app
  /routers
    users.py
    protected.py
  /models
    models.py
  /schemas
    schemas.py
  /core
    auth.py
    config.py
    dependencies.py
  database.py
  main.py
/tests
  test_auth.py
  test_users.py
  test_protected.py
```

## 13. Piano di implementazione
1. Definizione schema database (users, reset tokens).
2. Setup ambiente FastAPI e configurazione PostgreSQL.
3. Implementazione modelli ORM e migrazioni DB.
4. Sviluppo endpoint registrazione con validazione + hash password.
5. Sviluppo endpoint login con verifica credenziali e generazione JWT.
6. Creazione dipendenza FastAPI per validazione token JWT.
7. Implementazione endpoint protetti di test (/protected).
8. Realizzazione feature reset password: richiesta e conferma.
9. Stesura e esecuzione test unitari e integrazione.
10. Verifica automazione documentazione API via OpenAPI/Swagger.
11. Deployment ambiente di test per verifica endpoint.