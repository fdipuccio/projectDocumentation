MODULE: BACKEND VERSION: 1
## 1. Obiettivo backend
Progettare e sviluppare un backend API sicuro, scalabile e manutenibile basato su FastAPI e PostgreSQL, che gestisca l’autenticazione utenti tramite email e password con i seguenti servizi principali: registrazione con validazione univocità email e hashing password, login con generazione token JWT per sessioni, protezione degli endpoint tramite verifica JWT, funzionalità base di reset password tramite token temporaneo inviato via email, con gestione di errori RESTful e documentazione API automatica.


## 2. Assunzioni tecniche
- Solo email e password come dati utente essenziali; email deve essere univoca.
- Password con policy minima definita (es. lunghezza almeno 8, presenza di lettere e numeri, caratteri speciali opzionali) e hashing sicuro con bcrypt.
- Token JWT con scadenza definita ma nessun meccanismo di revoca o blacklist implementato in MVP.
- Reset password tramite token temporaneo memorizzato in DB con scadenza breve (es. 1 ora).
- Invio email reset tramite sistema SMTP configurabile, con fallback placeholder in ambiente di sviluppo.
- Nessuna integrazione esterna (OAuth, 2FA, CAPTCHA), nessuna gestione ruoli/autorizzazioni granulari.
- Validazione input rigorosa per evitare leak dati sensibili e injection.
- Ambiente Python, FastAPI e PostgreSQL stabile e disponibile.
- Architettura modulare per futura estendibilità (logging, policy più robuste, revoca token).


## 3. Architettura backend
Architettura a componenti modulari a 3 livelli principali:
- **API Layer / Controller:** gestisce routing HTTP, validazione base dati di input/output, chiamate ai servizi business e ritorno delle risposte RESTful.
- **Business Logic Layer:** implementa regole di autenticazione, validazione avanzata, generazione token JWT, gestione reset password, hash verifica password.
- **Persistence Layer:** interfaccia con PostgreSQL tramite ORM (es. SQLAlchemy) per persistenza utenti e token reset.
Supporto di middleware/dipendenze FastAPI per autenticazione JWT lato API Layer. Componenti separati per invio email e gestione configurazione SMTP. OpenAPI generata automaticamente da FastAPI.


## 4. Moduli e responsabilità
- **models:** definizione schema DB (User, PasswordResetToken).
- **schemas:** Pydantic model per input/output API (Register, Login, ResetPasswordRequest, ResetPasswordConfirm).
- **database:** configurazione connessione DB e sessioni ORM.
- **security:** hashing password (bcrypt), generazione/verifica JWT, validazione password policy.
- **auth:** endpoint registrazione, login, dipendenza protezione JWT, generazione token reset.
- **password_reset:** logica generazione token reset, invio email tramite SMTP, conferma reset.
- **api:** routing e controller endpoint, gestione errori HTTP.
- **config:** configurazione ambiente (DB, SMTP, JWT secret, policy password).
- **tests:** test unitari e integrazione, mocking email.
- **utils:** helper comuni, esempi logging base.
- **main:** avvio FastAPI con montaggio router e middleware.


## 5. API principali
- **POST /register**  
  Registra utente nuovo. Richiede email (formato valido, unica) e password (rispetta policy).  
  Risposte: 201 Created o 409 Conflict (email esistente), 400 Bad Request (dati errati).

- **POST /login**  
  Autentica con email e password, restituisce token JWT in caso di successo.  
  Risposte: 200 OK con JWT, 401 Unauthorized credenziali errate.

- **GET /protected-example**  
  Endpoint esempio protetto da autenticazione JWT.  
  Risposte: 200 OK se autenticato, 401/403 se token assente o invalido.

- **POST /password-reset/request**  
  Richiedi reset password tramite invio email con token.  
  Input: email utente registrata.  
  Risposte: 200 OK anche se email non esiste (non leak info), 400 Bad Request se input errato.

- **POST /password-reset/confirm**  
  Conferma reset password con token. Input: token reset e nuova password.  
  Risposte: 200 OK se reset andato a buon fine, 400 Bad Request token invalido/scaduto, 404 Not Found token non trovato.

Tutte le risposte utilizzano status code HTTP appropriati e payload JSON standardizzati (es. {"detail": "..."}).


## 6. Business logic
- Registrazione: verifica email univoca, validazione password policy, hashing password con bcrypt saltato.
- Login: verifica esistenza utente e password hash, genera token JWT firmato con payload minimo (user_id, exp).
- JWT: implementa scadenza breve (es. 30 minuti), firma con segreto configurabile.
- Protezione endpoint: dipendenza FastAPI che estrae token dalla header Authorization Bearer, verifica firma e scadenza, restituisce 401/403 in caso di errori.
- Reset password: genera univoco token criptograficamente sicuro, memorizzato associato utente con scadenza (es. 1 ora). Invio email con link/token contenente token. Validazione token conferma reset per sovrascrivere password dopo validazione policy.
- Validazione input estesa per protezione contro injection e data leak.
- Conservazione token reset con flag di validità (una sola volta utilizzabile), meccanismo di pulizia periodica con task futuro.
- Logging minimo degli eventi critici (registrazione fallita, login fallito, reset inviato).


## 7. Persistenza e integrazioni
- PostgreSQL come DB relazionale principale.
- Tabelle:
  - **users:** id PK, email unico, password_hash, data_creazione, eventuali campi di auditing base.
  - **password_reset_tokens:** id PK, user_id FK, token, expires_at, used_flag, data_creazione.
- ORM SQLAlchemy (o equivalente) per astrazione DB.
- Configurazione DB gestita tramite file/environment variabili.
- Servizio SMTP configurabile tramite parametri ambiente (host, porta, user, password, TLS).
- In ambienti di sviluppo placeholder per invio email (log o mock).
- Nessun sistema esterno aggiuntivo previsto.


## 8. Autenticazione e autorizzazione
- Autenticazione basata su JWT: token firmato con segreto forte, expiry predefinito.
- Password memorizzate con bcrypt salted hash; verifica login con confronto hash.
- Nessuna gestione ruoli o permessi, autorizzazione basata solo su presenza e validità token JWT.
- Middleware/dipendenza FastAPI per protezione endpoint; token recuperato da header Authorization.
- Reset password realizza bypass login standard con verifica token reset.
- Nessuna revoca token JWT all’interno del MVP, future estensioni suggerite.
- Controlli preventivi su input e risposta per evitare leak dati di autenticazione.


## 9. Gestione errori
- Risposte HTTP coerenti e esplicative:
  - 400 Bad Request: input malformato o dati non validi.
  - 401 Unauthorized: token JWT assente o invalido.
  - 403 Forbidden: token valido ma tentativo accesso non autorizzato (minimo attuale).
  - 404 Not Found: risorse/token non trovati (es. reset token).
  - 409 Conflict: email già esistente in registrazione.
  - 500 Internal Server Error: errori inattesi server.
- Messaggi dettagliati ma privi di informazioni sensibili.
- Validazione input con Pydantic e gestione eccezioni FastAPI.
- Logging minimale di errori a scopo diagnostico (estendibile).
- Fallback in caso di problemi invio email con risposta REST simulata in ambiente dev.


## 10. Strategia di test backend
- **Unit test:**
  - Funzioni hashing/verifica password.
  - Generazione/verifica token JWT con casi limite (scadenza, payload).
  - Validazione dati email/password (inclusi formati errati).
  - Token reset generazione/validazione e marchiatura uso.
- **Test integrazione:**
  - Endpoints registrazione (successo, email duplicata).
  - Login con credenziali corrette e errate.
  - Accesso a endpoint protetto con JWT valido, invalido, assente.
  - Invio reset password per email esistente e non.
  - Conferma reset password con token valido, scaduto, invalido.
- **Test sicurezza base:**
  - Verifica password non salvate o trasmesse in chiaro.
  - Prevenzione leak dati sensibili in messaggi errore.
- **Mocking invio email SMTP per automatizzazione test.
- Test di carico minimale su DB per validare connessioni persistenti.
- Verifica documentazione API aggiornata e corretta con OpenAPI.
- Copertura test garantita per modulistica e principale flussi critici.


## 11. Rischi tecnici
- Mancanza di definizione avanzata password policy può condurre a debolezze;
- Assenza di revoca token JWT limita possibilità rimozione sessioni compromesse;
- Funzionalità reset password dipende da sistema SMTP, configurazione multi-ambiente non uniformata;
- Possibile accumulo token reset nel DB senza meccanismo pulizia automatica attivo;
- Vulnerabilità brute force login non mitigata (out of scope, ma da considerare);
- Potenziale leak informazioni o errori nell’implementazione di validazioni e gestione token;
- Mancata gestione fallback o retry sistemi invio email può compromettere esperienza utente;


## 12. Struttura file proposta
```
/app
  /api
    __init__.py
    endpoints.py       # definizione router e controller endpoint
  /auth
    __init__.py
    auth_service.py    # login, generazione JWT, protezione endpoint
  /models
    __init__.py
    user.py            # schema DB user e reset token
  /schemas
    __init__.py
    user.py            # Pydantic schemas input/output
  /security
    __init__.py
    hashing.py         # bcrypt encode/verify, password policy
    jwt_handler.py     # generazione e verifica JWT
  /password_reset
    __init__.py
    service.py         # generazione token reset, conferma reset
    email_service.py   # invio email SMTP o placeholder
  /database
    __init__.py
    session.py         # configurazione connessione DB, ORM base
  /config
    __init__.py
    settings.py        # configurazioni ambiente (JWT secret, SMTP, DB)
  /utils
    helpers.py         # funzioni helper comuni e logging base
  /tests
    test_auth.py
    test_password_reset.py
    test_models.py
    test_api.py
  main.py              # avvio FastAPI, montaggio router, middleware
```

## 13. Piano di implementazione
1. Definizione e implementazione schema dati utenti e token reset su PostgreSQL.
2. Setup FastAPI con connessione DB configurata e test connessione.
3. Implementazione modulo sicurezza (hashing password, JWT generation).
4. Sviluppo endpoint registrazione con validazione e hashing.
5. Sviluppo endpoint login con verifica password e generazione token JWT.
6. Realizzazione dipendenza/middleware per protezione endpoint JWT.
7. Endpoint protetto di verifica accesso (es /protected-example).
8. Implementazione base funzionalità reset password: generazione token, persistenza, invio email (SMTP o placeholder).
9. Endpoint invio email reset e conferma reset password con validazione token.
10. Implementazione validazioni input, policy password minima.
11. Gestione errori centralizzata con risposte RESTful.
12. Scrittura test unitari e integrazione per tutte le funzioni e endpoint chiave, con mocking email.
13. Generazione e verifica documentazione OpenAPI/Swagger automatizzata da FastAPI.
14. Predisposizione configurazioni ambiente per multi-setup (dev/prod).
15. Stesura documentazione interna di policy sicurezza e flussi token.
16. Review sicurezza e pulizia codice pre-release.

Azioni successive suggerite:
- Implementazione meccanismo di revoca JWT futuro.
- Policy di pulizia periodica token reset scaduti (job separato).
- Potenziamento log e monitoring eventi critici.
- Definizione e applicazione policy password più dettagliate.
- Gestione limitazioni tentativi login e eventuale introduzione sicurezza avanzata.

La soluzione così definita garantisce una base solida, sicura e scalabile conforme ai requisiti MVP, con margine di evoluzione futura.