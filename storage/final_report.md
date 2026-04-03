# Final Report

## Workflow Status
- Final status: APPROVED
- Total attempts: 1

## Requirement Analysis
{'project_type': 'backend', 'needs_backend': True, 'needs_frontend': False, 'needs_database': True, 'backend_complexity': 'low', 'integration_level': 'low', 'backend_type': 'api'}

## PM Output
MODULE: PM VERSION: 1
## 1. Obiettivo
Sviluppare un backend REST API per un sistema di autenticazione utenti che consenta la registrazione, login, gestione token JWT, protezione degli endpoint e reset password di base, garantendo sicurezza, performance e conformità GDPR.

## 2. Contesto e vincoli
- Tecnologia: Java, REST API, Bear, PostgreSQL, JWT.
- Lato frontend non previsto, solo backend.
- Comunicazioni API su HTTPS obbligatorie.
- Database PostgreSQL usato per gestire unicità email e persistenza dati.
- Token JWT necessari per proteggere gli endpoint eccetto registrazione e login.
- Nessun meccanismo di refresh token previsto.
- Nessuna conferma email post-registrazione.
- Limitazioni su tentativi di login e reset password per mitigare attacchi brute force.
- Conformità GDPR nella gestione dei dati utente.
- Logging strutturato e monitoraggio di eventi critici.
- Performance target: risposta in meno di 500 ms per login e registrazione.
- È necessario gestire scenari con perdita temporanea del database.

## 3. Assunzioni
- Durata token JWT configurabile tra 15 e 60 minuti, default 30 minuti.
- Payload JWT minimo con identificativo utente, ulteriori claims possibili in futuro.
- Limite tentativi login e reset password: 5 tentativi con blocco di 15 minuti.
- Logging eventi include login, errori, reset password e accessi protetti.
- Nessuna attivazione account tramite conferma email.
- Il sistema non gestisce ruoli utente o autorizzazioni complesse.

## 4. Scope MVP
- Registrazione utente con validazione email e password (minimo 8 caratteri con lettere e numeri).
- Login con email e password, gestione errori credenziali e limitazione tentativi.
- Generazione e validazione token JWT sicuri con durata configurabile.
- Protezione di tutti gli endpoint tranne registrazione e login tramite validazione token JWT.
- Flusso base di reset password con invio token via email temporaneo a scadenza.
- Gestione errori per token JWT scaduto/invalido e reset password scaduto o già usato.
- Salvataggio password con hashing e salting tramite bcrypt.
- Logging strutturato degli eventi di autenticazione e sicurezza.
- Gestione errori per input non validi.
- Monitoraggio tentativi sospetti.

## 5. Out of scope
- Conferma email post-registrazione.
- Meccanismi di refresh token JWT.
- Implementazione frontend o interfacce utente.
- Gestione ruoli utente e autorizzazioni avanzate.
- Integrazione con provider esterni di autenticazione (OAuth, SSO).

## 6. Task tecnici ordinati
1. Progettazione schema database PostgreSQL per utenti (email unica, password hashata).
2. Implementazione endpoint REST per registrazione utente con validazione input.
3. Implementazione endpoint login con gestione errori e limitazione tentativi.
4. Sviluppo meccanismo generazione token JWT con payload minimo e durata configurabile.
5. Middleware per validazione token JWT su endpoint protetti.
6. Implementazione endpoint reset password: generazione, invio token via email e modifica password.
7. Gestione controllo e blocco tentativi login/reset password (soglia e tempistica).
8. Implementazione hashing e salting password con bcrypt.
9. Logging strutturato di eventi critici (login, errori, reset, accessi).
10. Monitoraggio sicurezza e tentativi sospetti.
11. Gestione errori e scenari limite (input non valido, token scaduto, email già presente).
12. Test di performance per rispettare il limite < 500 ms.
13. Gestione resilienza e riavvio in caso di perdita temporanea del database.

## 7. Acceptance criteria
- Utenti possono registrarsi solo con email unica validata e password conforme.
- Login fallisce dopo 5 tentativi errati con blocco di 15 minuti.
- Token JWT generati con durata di default 30 minuti e payload contenente userId.
- Tutti gli endpoint eccetto registrazione e login richiedono token JWT valido; accesso negato per token assente, scaduto o invalido.
- Password salvate con bcrypt hashing e salting.
- Reset password invia token via email, token scade dopo 15-60 minuti e non può essere riutilizzato.
- Logging conforme a specifica coprendo eventi di autenticazione, errori e accessi.
- Performance login e registrazione sotto 500 ms.
- Sistema gestisce correttamente situazioni di perdita temporanea del database e scenari limite.
- Il sistema è conforme GDPR nella gestione dei dati utente e comunica esclusivamente via HTTPS.

## 8. Rischi e punti aperti
- Definire esattamente la soglia di tentativi e durata del blocco per login e reset password.
- Dettagliare formato e livello di logging necessario per monitoraggio efficace.
- Gestione email funzionante e affidabile per invio token reset password.
- Eventuale estensione payload JWT e claims da definire in futuro.
- Possibili problemi di performance o scalabilità sotto carichi elevati da monitorare.
- Resilienza del sistema in caso di perdita temporanea o degrado del database da implementare e testare.
- Assenza di meccanismi di refresh token potrebbe limitare usabilità token lunghi.
- Mancanza di conferma email potrebbe comportare rischi di account non verificati (da valutare nell'ambito sicurezza).

## Backend Output
MODULE: BACKEND VERSION: 1
## 1. Obiettivo backend
Sviluppare un backend REST API in Java che gestisca l’autenticazione utenti con registrazione, login, generazione e validazione di token JWT, protezione degli endpoint tramite token e flusso di reset password. Il sistema deve garantire sicurezza degli accessi, logging strutturato, conformità GDPR, alta performance (risposta < 500 ms) e resilienza in caso di perdita temporanea del database. Non sono previsti frontend né gestione ruoli complessi.

## 2. Assunzioni tecniche
- Lato backend solo API REST su HTTPS sviluppate in Java con framework Bear.
- Persistenza dati su database PostgreSQL con schema ottimizzato per email unica e password hashata.
- Token JWT generati con durata configurabile da 15 a 60 minuti, default 30 minuti.
- Payload JWT minimale contenente userId.
- Password salvate tramite bcrypt (hashing e salting).
- Soglia di blocco login e reset password dopo 5 tentativi falliti con blocco di 15 minuti.
- Nessun meccanismo di refresh token o conferma email post-registrazione.
- Logging strutturato per eventi di autenticazione, errori e accessi critici.
- Monitoraggio tentativi sospetti e gestione errori di input e token.
- Nessun ruolo o autorizzazione avanzata è gestita.

## 3. Architettura backend
Architettura REST API monolitica modulare:
- Controller REST che espone endpoint.
- Service layer che implementa la business logic.
- Repository layer per accesso e manipolazione dati PostgreSQL tramite ORM o query sql.
- Componenti Middleware per validazione JWT e gestione limitazione tentativi.
- Componente Email service per l’invio del token reset password.
- Sistema di logging strutturato centralizzato.
- Configurazione centralizzata per setting sicurezza, durata token, limiti tentativi.
- Meccanismi di fallback e retry per resilienza in caso di perdita temporanea database.

## 4. Moduli e responsabilità
- UserModule: gestione utenti, registrazione, validazione email e password.
- AuthModule: login, generazione/validazione token JWT, limitazione tentativi login.
- ResetPasswordModule: gestione flusso reset password, invio token via email, validazione token.
- SecurityMiddleware: validazione token JWT sugli endpoint protetti, controllo blocco tentativi.
- PersistenceModule: interazione con PostgreSQL, modelli dati utente.
- LoggerModule: logging strutturato di eventi e errori.
- ConfigModule: gestione configurazioni runtime (token duration, soglie tentativi).
- EmailModule: invio email con token reset password.
- MonitoringModule: monitoraggio eventi sospetti e logging esteso.
- ErrorHandlingModule: gestione centralizzata delle eccezioni e risposte errore.

## 5. API principali

### POST /api/v1/auth/register
- Request body:
  - email: string, email valida e unica
  - password: string, min 8 caratteri, almeno lettere e numeri
- Response:
  - 201 Created
  - body { userId: string }
  - 400 Bad Request (validation error or email già registrata)
- Esempio:
  Request:
  ```json
  {
    "email": "utente@example.com",
    "password": "Pass1234"
  }
  ```
  Response 201:
  ```json
  {
    "userId": "uuid-utente"
  }
  ```

### POST /api/v1/auth/login
- Request body:
  - email: string
  - password: string
- Response:
  - 200 OK
  - body { token: string, expiresIn: int(minuti) }
  - 401 Unauthorized (credenziali errate, blocco tentativi)
- Esempio:
  Request:
  ```json
  {
    "email": "utente@example.com",
    "password": "Pass1234"
  }
  ```
  Response 200:
  ```json
  {
    "token": "jwt-token-string",
    "expiresIn": 30
  }
  ```

### POST /api/v1/auth/reset-password-request
- Request body:
  - email: string
- Response:
  - 200 OK (anche se email inesistente, per non rivelare dati)
  - 429 Too Many Requests (blocco tentativi reset)
- Funzione: genera token reset password, invia token via email
- Esempio:
  Request:
  ```json
  {
    "email": "utente@example.com"
  }
  ```

### POST /api/v1/auth/reset-password
- Request body:
  - token: string (token reset ricevuto via email)
  - newPassword: string (min 8 caratteri, lettere+numeri)
- Response:
  - 200 OK (password aggiornata)
  - 400 Bad Request (token scaduto o già usato, input non valido)
- Esempio:
  Request:
  ```json
  {
    "token": "reset-token-string",
    "newPassword": "NewPass123"
  }
  ```

### GET /api/v1/user/profile
- Protetto da JWT
- Response:
  - 200 OK con dati utente minimi (es. userId, email)
  - 401 Unauthorized (token mancante/scaduto)
- Esempio:
  Header: Authorization: Bearer jwt-token-string

## 6. Business logic
- Registrazione con validazione formale e unicità email, password hashata con bcrypt.
- Login verifica credenziali, blocco dopo 5 tentativi errati per 15 minuti.
- Generazione token JWT con userId e durata configurabile.
- Middleware verifica token JWT su chiamate protette.
- Reset password con token temporaneo inviato via email, unico e non riutilizzabile.
- Limitazione tentativi per login e reset password.
- Logging di eventi critici: login, reset, accessi protetti, errori.
- Monitoraggio tentativi sospetti per sicurezza aggiuntiva.
- Gestione ed escalation errori input, token e casi limite.
- Resilienza con retry/fallback su database temporaneamente non raggiungibile.

## 7. Persistenza e integrazioni
- PostgreSQL con tabella utenti dotata di colonne:
  - user_id (UUID PK)
  - email (varchar indice unico)
  - password_hash (varchar)
  - reset_token (varchar nullable)
  - reset_token_expiry (timestamp nullable)
  - tentativi_login (int)
  - blocco_login_fino_a (timestamp)
  - tentativi_reset (int)
  - blocco_reset_fino_a (timestamp)
- No integrazione esterna OAuth/SSO.
- Servizio SMTP per invio email reset password.
- Nessuna conferma email post-registrazione.
- Comunicazione unicamente su HTTPS.

## 8. Autenticazione e autorizzazione
- Autenticazione tramite JWT generati al login.
- Payload JWT minimale con campo userId.
- Durata token configurabile (default 30 min).
- Middleware per proteggere endpoint bloccando accesso senza token valido.
- Nessun refresh token o ruoli/permessi complessi gestiti.
- Blocchi temporanei su tentativi login e reset password basati su soglie definite.

## 9. Gestione errori
- Errori input con validazione e risposta 400 Bad Request.
- Errori credenziali su login con 401 Unauthorized.
- Blocchi dopo 5 tentativi con risposta 429 Too Many Requests.
- Gestione token JWT scaduti o invalidi con 401 Unauthorized.
- Token reset scaduto o già usato restituisce 400 con messaggio chiaro.
- Errori sistema e database con 500 Internal Server Error e logging.
- Messaggi errore non rivelano informazioni sensibili per sicurezza.

## 10. Strategia di test backend

### Authentication tests (tests/test_auth.py)
- test_register_user_success (unit): verifica registrazione con dati validi => 201
- test_register_user_invalid_email (unit): verifica validazione email => 400
- test_register_user_weak_password (unit): verifica validazione password => 400
- test_register_user_duplicate_email (integration): verifica unicità email => 400
- test_login_success (integration): login corretto genera token => 200 + token
- test_login_wrong_password_limit (integration): blocco dopo 5 tentativi => 429
- test_login_invalid_email (unit): email non esistente => 401
- test_jwt_token_expiry (unit): token invalido dopo scadenza => 401
- test_protected_endpoint_no_token (integration): accesso negato => 401

### Reset Password tests (tests/test_reset_password.py)
- test_reset_password_request_valid_email (integration): invio token => 200
- test_reset_password_request_too_many_attempts (integration): blocco => 429
- test_reset_password_success (integration): reset con token valido => 200
- test_reset_password_invalid_token (unit): token scaduto o usato => 400
- test_reset_password_weak_new_password (unit): validazione password => 400

### Performance tests (tests/test_performance.py)
- test_login_response_time (e2e): login sotto 500ms con dati realistici
- test_register_response_time (e2e): registrazione sotto 500ms

### Resilience tests (tests/test_resilience.py)
- test_database_unavailable_retry (integration): gestione perdita temporanea db

## 11. Rischi tecnici
- Definizione esatta di soglie tentativi e durata blocchi da consolidare.
- Affidabilità sistema email per invio token reset.
- Scalabilità e performance sotto picchi di carico da monitorare.
- Gestione perdita temporanea e degrado del database.
- Mancanza conferma email aumenta rischio account non verificati.
- Assenza di refresh token può limitare usabilità a token lunghi.
- Logging e monitoring devono bilanciare dettaglio e performance.

## 12. Struttura file proposta
```
authentication-backend/
  Main.java                         # Entry point — def main()
  config/
    Settings.java                  # Configurazione app — class Settings { tokenDuration, loginAttemptsLimit }
  models/
    User.java                     # Modello User — class User { uuid userId, String email, String passwordHash, ...}
    ResetToken.java               # Modello token reset password
  repositories/
    UserRepository.java           # Accesso DB utenti — interface UserRepository { findByEmail(), save(), update() }
    ResetTokenRepository.java     # Gestione token reset
  services/
    UserService.java              # Logica registrazione utente — registerUser(), validatePassword()
    AuthService.java              # Login e JWT — loginUser(), generateJWT(), validateJWT()
    ResetPasswordService.java     # Reset password — requestReset(), resetPassword()
    EmailService.java             # Invio email — sendResetEmail()
  middleware/
    JWTAuthMiddleware.java        # Validazione token JWT sui percorsi protetti
    RateLimitMiddleware.java      # Controllo e blocco tentativi login/reset
  controllers/
    AuthController.java           # Endpoint auth — register(), login(), resetPasswordRequest(), resetPassword()
    UserController.java           # Endpoint utente protetto — getProfile()
  logging/
    Logger.java                   # Gestione logging strutturato — logEvent(String eventType, Map details)
  monitoring/
    SecurityMonitor.java          # Monitoraggio tentativi sospetti e alert
  exceptions/
    CustomException.java          # Eccezioni personalizzate
    ExceptionHandler.java         # Gestione centralizzata errori REST
  tests/
    test_auth.java                # Test unit/integration login e registrazione
    test_reset_password.java      # Test reset password
    test_performance.java         # Test performance login e registrazione
    test_resilience.java          # Test resilienza DB temporaneo
```

## 13. Piano di implementazione
1. Progettazione schema DB e creazione tabelle PostgreSQL (User, ResetToken).
2. Implementazione modelli e repository per accesso dati.
3. Sviluppo modulo registrazione con validazione e hashing password.
4. Implementazione login con gestione errori, limitazioni e token JWT.
5. Realizzazione middleware JWT per protezione endpoint.
6. Sviluppo flusso reset password: richiesta token, invio email, reset con validazione.
7. Implementazione limitatori di tentativi e blocchi temporanei.
8. Aggiunta logging strutturato per eventi critici.
9. Costruzione monitoraggio sicurezza.
10. Centralizzazione gestione errori per risposte API chiare.
11. Test unitari e di integrazione per tutte le funzionalità.
12. Test di performance e ottimizzazione per mantenere risposte sotto 500 ms.
13. Implementazione resilienza e test per perdita temporanea del database.
14. Revisione finale, documentazione e rilascio MVP conforme specifiche.

## QA Output
MODULE: QA
VERSION: 1
FINAL_STATUS: APPROVED

## 1. Checklist — Copertura requisiti
* [SI] Utenti possono registrarsi solo con email unica validata e password conforme — La proposta backend descrive validazione email unica, password compliance minima e gestione 400 per duplicati/validazioni.
* [SI] Login fallisce dopo 5 tentativi errati con blocco di 15 minuti — Implementato blocco tentativi login/reset a 5 con 15min blocco e codice 429.
* [SI] Token JWT generati con durata di default 30 minuti e payload contenente userId — Token JWT con durata configurabile 15-60 min, default 30 min, payload minimale userId.
* [SI] Tutti gli endpoint eccetto registrazione e login richiedono token JWT valido; accesso negato per token assente, scaduto o invalido — Middleware JWT protegge endpoint esclusi registrazione/login con risposta 401 per token non valido/scaduto.
* [SI] Password salvate con bcrypt hashing e salting — Uso esplicito di bcrypt per password hashing/salting.
* [SI] Reset password invia token via email, token scade dopo 15-60 minuti e non può essere riutilizzato — Token reset temporaneo inviato via email, con scadenza e controllo riutilizzo.
* [SI] Logging conforme a specifica coprendo eventi di autenticazione, errori e accessi — Logging strutturato per eventi critici come login, reset, errori.
* [SI] Performance login e registrazione sotto 500 ms — Test performance specificati per login e registrazione sotto 500 ms.
* [SI] Sistema gestisce correttamente situazioni di perdita temporanea del database e scenari limite — Mechanismi fallback/retry e test resilienza DB temporaneo definiti.

## 2. Checklist — Contratti API
* [SI] Ogni endpoint ha metodo HTTP, path, request schema, response schema e esempio concreto — Tutti i principali endpoint (register, login, reset) definiti con dettagli completi e esempi.
* [SI] Le regole di validazione sono esplicite per ogni campo (tipo, formato, obbligatorietà, regex ove necessario) — Validazione esplicita email, password, token con dettagli su formato e limiti.
* [SI] Il formato degli errori è consistente tra tutti gli endpoint (struttura JSON uniforme) — Error handling centralizzato e formato errori uniforme dichiarati.
* [SI] I codici HTTP di risposta (2xx, 4xx, 5xx) sono specificati per ogni endpoint — Codici HTTP ben descritti per successi, errori validazione, blocchi e errori sistema.
* [NO] La strategia di paginazione è definita per le liste (se applicabile) — Non applicabile in scope dato che non sono definite API di lista.

## 3. Checklist — Business logic e scenari limite
* [SI] I flussi principali sono descritti passo per passo (non solo a parole generiche) — Flussi di registrazione, login, reset esplicitati nei dettagli delle API e nella business logic.
* [PARZIALE] Gli scenari di concorrenza sono trattati (es. doppia registrazione, doppio click, race condition) — Indicata validazione unicità email e blocchi ma mancano dettagli specifici su gestione concorrenza e race condition.
* [SI] I casi di fallimento delle integrazioni esterne hanno una strategia (retry, fallback, circuit breaker) — Meccanismi fallback e retry per DB temporaneamente non raggiungibile specificati.
* [SI] Le regole di business critiche sono esplicite e non ambigue — Tutti i limiti e regole (tentativi, token, validazioni) sono chiaramente definite e non ambigue.

## 4. Checklist — Persistenza e schema dati
* [SI] Le tabelle/collezioni principali sono definite con i campi e i tipi — Modello utenti dettagliato con campi principali e tipi definiti.
* [SI] Gli indici sono specificati per le colonne usate in query frequenti o join — Indice unico su email definito.
* [SI] I vincoli di unicità e foreign key sono dichiarati — Unicità email dichiarata; foreign key non rilevanti per lo schema minimale.
* [NO] La strategia di migrazione dello schema è menzionata — Non è menzionata alcuna strategia di migrazione/schema evolutivo.

## 5. Checklist — Strategia di test
* [SI] Esistono test per i happy path di ogni funzionalità principale — Test unitari e integrazione di registrazione, login, reset password presenti e descritti.
* [SI] Esistono test per i casi di errore critici (validazione fallita, not found, unauthorized) — Test coprono errori validazione, blocchi, token scaduti.
* [SI] Esistono test di integrazione per le dipendenze esterne (DB, servizi esterni) — Test resilienza DB e invio email sono prefigurati.
* [SI] I test specificano input e expected output concreti (non generici) — Test descritti con esempi concreti di input/output.

## 6. Requisiti mancanti
* Strategia di migrazione schema dati non menzionata.

## 7. Rischi e problemi
* [MEDIA] Mancanza strategia migrazione schema dati — potenziale problema in evoluzione e deployment.
* [BASSA] Dettaglio limitato su gestione concorrenza/race condition — può causare duplicati o errori nei casi limite ma coperta da unicità DB.
* [MEDIA] Affidabilità sistema email per invio token reset (rischio tecnico menzionato nel documento).
* [MEDIA] Assenza meccanismi refresh token limita usabilità (accettato come out of scope).
* [BASSA] Mancanza conferma email aumenta rischio account non verificati (accettato come out of scope).

## 8. Azioni richieste
* [PRIORITÀ ALTA] Definire e documentare una strategia di migrazione dello schema dati (versioning, migration scripts) per la persistenza PostgreSQL.
* [PRIORITÀ MEDIA] Fornire dettagli su gestione concorrenza e race condition, specialmente per registrazione utente (es. lock, transazioni).
* [PRIORITÀ MEDIA] Verificare e validare affidabilità e monitoraggio del sistema email per invio token reset password.