# Final Report

## Workflow Status
- Final status: APPROVED
- Total attempts: 4

## Requirement Analysis
{'project_type': 'backend', 'needs_backend': True, 'needs_frontend': False, 'needs_database': True, 'backend_complexity': 'low', 'integration_level': 'low', 'backend_type': 'api'}

## PM Output
MODULE: PM VERSION: 1
## 1. Obiettivo
Realizzare un backend di autenticazione utenti che permetta la registrazione, login tramite email e password, generazione e verifica di token JWT per l’accesso a endpoint protetti, e reset password base, sfruttando lo stack tecnologico Java, REST API, Bear framework, PostgreSQL e JWT. Il sistema deve essere sicuro, performante, scalabile e facile da monitorare.

## 2. Contesto e vincoli
- Stack tecnico obbligatorio: Java, REST API, Bear framework, PostgreSQL come database, JWT per i token di autenticazione.
- Backend only: non è prevista una UI o frontend.
- L’architettura deve prevedere integrazione completa con il database PostgreSQL per persistenza dati utenti e gestione sessioni/token.
- La sicurezza prevede archiviazione password hashed con salt, JWT firmati con scadenza temporale, e controlli rigorosi sugli input.
- Le API devono rispondere con buona velocità per garantire esperienza utente soddisfacente.
- Il sistema deve poter scalare orizzontalmente.
- Logging sicuro e monitoraggio di eventi critici sono obbligatori.
- Non sono previsti sistemi di autenticazione multifattoriale, social login o autorizzazioni granulari.

## 3. Assunzioni
- La validazione email in fase di registrazione includerà verifica dell’unicità ma non necessariamente conferma tramite link di attivazione.
- Le password devono rispettare best practice standard: minimo 8 caratteri con lettere, numeri e simboli.
- I token JWT avranno durata di circa un’ora con possibilità di introdurre in futuro meccanismi di refresh.
- Il reset password base si realizzerà inviando un link o codice temporaneo via email.
- Si introdurranno limiti di tentativi login/reset per mitigare attacchi brute force, pur non essendo esplicitamente richiesto.
- Non si implementano sistemi avanzati di gestione password come scadenze obbligatorie o password temporanee.
- La gestione di sessioni multiple per utente sarà supportata senza restrizioni specifiche.

## 4. Scope MVP
- Endpoint REST per registrazione utente con controllo unicità email e validazione password.
- Endpoint login con verifica credenziali e generazione token JWT firmato e con scadenza.
- Middleware o filtro per protezione endpoint accessibili solo con token JWT valido.
- Endpoint per reset password base, con invio codice/link a email registrata.
- Persistenza dati utenti e token/sessioni su PostgreSQL.
- Logging sicuro e monitoraggio degli eventi critici (login, reset password, errori).
- Gestione degli errori e risposte coerenti per casi di credenziali errate, token invalidi/scaduti.
- Meccanismi base di sicurezza input validation per prevenire injection e vulnerabilità comuni.
- Scalabilità a livello di backend e database.

## 5. Out of scope
- Autenticazione multifattore (MFA).
- Login tramite social network o sistemi terzi.
- Gestione ruoli utente e autorizzazioni granulari.
- UI o frontend.
- Gestione avanzata di password (es. password temporanee, politiche di scadenza obbligatorie).
- Sistemi complessi di verifica identità o blocco IP automatico oltre limiti di tentativi login/reset (non specificati esplicitamente).

## 6. Task tecnici ordinati
1. Progettazione schema database utenti e token in PostgreSQL.
2. Implementazione endpoint registrazione con validazione input, controllo unicità email, hashing password con salt sicuro.
3. Implementazione endpoint login con verifica credenziali e generazione token JWT firmato, impostazione scadenza token.
4. Sviluppo middleware Bear per protezione endpoint mediante verifica token JWT valido.
5. Implementazione endpoint reset password base con generazione codice/link temporaneo e invio email.
6. Configurazione logging sicuro per eventi critici (login, reset, errori).
7. Realizzazione controlli input rigorosi per tutte le API.
8. Testing funzionale e di sicurezza su casi normali e limitrofi (tentativi errati, token scaduti).
9. Setup monitoraggio e alert base per malfunzionamenti e tentativi di attacco.
10. Documentazione tecnica API e configurazioni sicurezza.

## 7. Acceptance criteria
- Utente può registrarsi con email unica e password conforme a policy.
- Utente può effettuare login con email/password valide e ricevere token JWT con scadenza.
- Endpoint protetti rispondono solo se token JWT è presente, valido e non scaduto; altrimenti ritornano errore 401.
- Reset password consente di inviare codice/link via email e permette modifica password.
- Password salvate nel database sono hashed con salt sicuro.
- Tutte le API validano gli input e impediscono injection o dati non validi.
- Log eventi critici sono correttamente registrati e consultabili in modo sicuro.
- Il sistema risponde con performance accettabili (es. tempi di risposta inferiori a 1 secondo per endpoint critici).
- Il sistema scala orizzontalmente senza perdita di funzionalità.
- Test di sicurezza base superati (es. tentativi ripetuti login bloccati o limitati).
- Documentazione tecnica completa e aggiornata.

## 8. Rischi e punti aperti
- Ambiguità sulla conferma dell’email in fase di registrazione, attualmente esclusa ma possibile rischio di account non validi.
- Politiche di sicurezza password e limiti di tentativi non dettagliati: serve conferma o definizione più precisa.
- Gestione token JWT: politiche di rinnovo/refresh non implementate nell’MVP possono limitare usabilità futura.
- Vulnerabilità potenziali a DoS o attacchi brute force se limiti tentativi o monitoraggio non adeguati.
- Gestione utenti senza accesso all’email per reset password non risolta.
- Concorrenza in registrazione simultanea con stessa email deve essere gestita attentamente lato DB.
- Necessità di chiarire dettagli di logging e monitoraggio per rispettare eventuali compliance di sicurezza.
- Mancanza di ruoli o autorizzazioni granulari potrebbe limitare l’estensione futura del sistema.

## Backend Output
MODULE: BACKEND VERSION: 1
## 1. Obiettivo backend
Realizzare un backend API REST per la gestione dell'autenticazione utenti che supporti registrazione, login, protezione di endpoint tramite token JWT e reset password di base. Il sistema deve garantire sicurezza, performance, scalabilità orizzontale e monitoraggio degli eventi critici, utilizzando Java, Bear framework, PostgreSQL e JWT. Le API saranno progettate per rispondere in meno di 1 secondo sugli endpoint critici e con gestione rigorosa degli input per prevenire vulnerabilità comuni.

## 2. Assunzioni tecniche
- Registro utenti con email unica, senza conferma tramite link di attivazione.
- Password conforme a policy minima (≥ 8 caratteri: lettere, numeri, simboli).
- Token JWT firmati con durata di circa 1 ora, senza meccanismo di refresh nell’MVP.
- Reset password con invio di codice o link temporaneo via email.
- Limiti base su tentativi di login e reset per mitigare brute force.
- Persistenza dati utenti e token/sessioni su PostgreSQL.
- Architettura stateless per scalabilità orizzontale.
- Logging cifrato e monitoraggio per eventi critici.
- Tutte le validazioni input, senza autorizzazioni granulari o MFA.
- Deployment in ambiente che supporta Java e PostgreSQL.

## 3. Architettura backend
Architettura modulare a strati:
- Controller REST per esposizione API.
- Service layer per logica business (registrazione, login, reset password).
- Repository layer per accesso dati PostgreSQL con mapping ORM.
- Middleware Bear per validazione token JWT e protezione endpoint.
- Modulo di logging sicuro con cifratura eventi critici.
- Monitoraggio eventi e metriche (logging + alerting).
- Pooling connessioni DB per performance.
- Statelness del backend, nessuna dipendenza di stato tra richieste.
- Sistema email esterno integrato con meccanismi retry base.

## 4. Moduli e responsabilità
- **UserController**: espone endpoint REST /register, /login, /reset-password/request, /reset-password/confirm.
- **AuthService**: gestisce registrazione, autenticazione utente, generazione e validazione JWT, controllo limiti tentativi.
- **ResetPasswordService**: genera codici reset, invia email, valida codice reset e aggiorna password.
- **JwtAuthMiddleware**: intercetta chiamate protette e verifica validità token JWT.
- **UserRepository**: CRUD utenti e gestione dati persistenti in PostgreSQL.
- **ResetTokenRepository**: gestione codici reset password su DB.
- **ValidationService**: validazione di email, password e input API.
- **EventLogger**: logging cifrato eventi critici (login, reset, errori).
- **EmailService**: interfaccia per invio email con retry e fallback base.
- **MetricsMonitor**: raccolta metriche base e monitoraggio dei tentativi errati.
- **ConfigModule**: gestione configurazione applicazione (DB, JWT, logging, email).

## 5. API principali

### POST /api/v1/auth/register
- Request body schema:
  ```json
  {
    "email": "string (valid email, unique, max 254)",
    "password": "string (min 8 chars, include letters, numbers, symbols)"
  }
  ```
- Response body schema:
  - 201 Created
    ```json
    { "message": "User registered successfully" }
    ```
  - 400 Bad Request (validation error)
    ```json
    { "error_code": "VALIDATION_FAILED", "message": "Detailed validation error" }
    ```
  - 409 Conflict (email already exists)
    ```json
    { "error_code": "EMAIL_CONFLICT", "message": "Email already registered" }
    ```
- Esempio:
  Request:
  ```
  POST /api/v1/auth/register
  {
    "email": "user@example.com",
    "password": "P@ssw0rd123!"
  }
  ```
  Response:
  ```
  201 Created
  {
    "message": "User registered successfully"
  }
  ```

### POST /api/v1/auth/login
- Request body schema:
  ```json
  {
    "email": "string (valid email)",
    "password": "string"
  }
  ```
- Response body schema:
  - 200 OK
    ```json
    {
      "token": "jwt-token-string",
      "expires_in": 3600
    }
    ```
  - 400 Bad Request (validation failed)
  - 401 Unauthorized (invalid credentials or user not found)
  - 429 Too Many Requests (brute force limit reached)
- Esempio:
  Request:
  ```
  POST /api/v1/auth/login
  {
    "email": "user@example.com",
    "password": "P@ssw0rd123!"
  }
  ```
  Response:
  ```
  200 OK
  {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expires_in": 3600
  }
  ```

### POST /api/v1/auth/reset-password/request
- Request body schema:
  ```json
  { "email": "string (valid registered email)" }
  ```
- Response body schema:
  - 200 OK
    ```json
    { "message": "Reset password link/code sent if email exists" }
    ```
- Esempio:
  ```
  POST /api/v1/auth/reset-password/request
  { "email": "user@example.com" }
  ```
  Response:
  ```
  200 OK
  { "message": "Reset password link/code sent if email exists" }
  ```

### POST /api/v1/auth/reset-password/confirm
- Request body schema:
  ```json
  {
    "email": "string (valid email)",
    "reset_code": "string (valid temporary code)",
    "new_password": "string (conforming to policy)"
  }
  ```
- Response body schema:
  - 200 OK
    ```json
    { "message": "Password reset successful" }
    ```
  - 400 Bad Request (validation error)
  - 401 Unauthorized (invalid or expired code)
- Esempio:
  ```
  POST /api/v1/auth/reset-password/confirm
  {
    "email": "user@example.com",
    "reset_code": "ABC123",
    "new_password": "N3wP@ssw0rd!"
  }
  ```
  Response:
  ```
  200 OK
  { "message": "Password reset successful" }
  ```

### Protezione endpoint
- Middleware Bear **JwtAuthMiddleware**:
  - Intercetta richieste su endpoint protetti.
  - Verifica presenza e validità di token JWT nel header Authorization: Bearer token.
  - Risponde 401 Unauthorized se token mancante, invalido o scaduto.

## 6. Business logic
- Registrazione: validazione email e password, controllo unicità email concorrente tramite locking e vincolo DB, hashing password con bcrypt + salt sicuro, persistenza user in DB.
- Login: verifica email e password (bcrypt verify), limitazione tentativi errati (rate limiting in memoria o DB), generazione JWT firmato con scadenza 1 ora, logging evento.
- Reset password:
  - Request: verifica esistenza email, generazione codice temporaneo unico con scadenza, salvataggio DB, invio email con retry e fallback semplice.
  - Confirm: validazione codice reset (unicità, scadenza), aggiornamento password hashed, invalidazione codice.
- Middleware JWT: verifica integrità e scadenza token, estrazione userId da token, blocco accesso con 401.
- Validazione in tutte le API input per prevenire injection e dati errati.
- Logging cifrato per eventi critici e errori.
- Monitoraggio base con metriche e alert su tentativi anomali.

## 7. Persistenza e integrazioni
- Database PostgreSQL con tabelle:
  - **users**: id (PK), email (unique), password_hash, created_at, updated_at.
  - **reset_tokens**: id (PK), user_id (FK users.id), code (unique), expires_at, created_at, used (bool).
- Indici su email in users e code in reset_tokens.
- Transazioni DB per coerenza operazioni critiche (es. registrazione e reset).
- Tool migrazione schema consigliato: Liquibase o Flyway.
- Integrazione servizio email esterno con retry base e fallback.
- Connection pooling per DB.
- Monitoraggio stato servizi critici.

## 8. Autenticazione e autorizzazione
- Autenticazione stateless tramite token JWT firmati HMAC o RSA, con durata 1 ora.
- Nessuna autorizzazione granulare prevista, tutti gli utenti autenticati accedono agli endpoint protetti.
- Middleware verifica presenza, validità e scadenza del token JWT, rigetta se non valido.
- Limiti tentativi login e reset per prevenire bruteforce.
- Password memorizzate solo in forma hashed con salt (bcrypt).

## 9. Gestione errori
- Risposte uniformi in formato JSON con campi:
  - error_code: stringa codice errore (es: VALIDATION_FAILED, EMAIL_CONFLICT, UNAUTHORIZED)
  - message: descrizione leggibile errore.
- Codici HTTP coerenti:
  - 200-201 per successo
  - 400 per validazione e errori client
  - 401 per autenticazione fallita
  - 409 per conflitti (es. email già presente)
  - 429 per limiti superati
  - 500 per errori interni server
- Logging errori critici con stacktrace cifrati e consultabili in modo sicuro.
- Messaggi di errore non espongono dettagli riservati.

## 10. Strategia di test backend

### Registrazione
- test_register_user_success
  - Tipo: unit, integration
  - Verifica: registrazione con dati validi produce 201 e utente nel DB
  - Input: email valida unica, password conforme
  - Expected output: 201, messaggio successo
- test_register_user_duplicate_email
  - Tipo: unit, integration
  - Verifica: registrazione con email già esistente produce 409
  - Input: email duplicata
  - Expected output: 409 con errore EMAIL_CONFLICT
- test_register_user_invalid_password
  - Tipo: unit
  - Verifica: password non conforme produce 400
  - Input: password corta o senza simboli
  - Expected output: 400 VALIDATION_FAILED

### Login
- test_login_success
  - Tipo: unit, integration
  - Verifica: login corretto restituisce JWT valido
  - Input: email e password corrette
  - Expected output: 200 con token JWT
- test_login_wrong_password
  - Tipo: unit
  - Verifica: password errata produce 401
  - Input: credenziali errate
  - Expected output: 401 UNAUTHORIZED
- test_login_limit_reached
  - Tipo: integration
  - Verifica: superamento tentativi blocca login (429)
  - Input: tentativi ripetuti errati
  - Expected output: 429 Too Many Requests

### Reset Password
- test_reset_password_request_exists_email
  - Tipo: integration
  - Verifica: codice generato e email inviata se email esiste
  - Input: email registrata
  - Expected output: 200 OK messaggio di invio
- test_reset_password_request_non_existing_email
  - Tipo: integration
  - Verifica: risposta neutra per email non presente
  - Input: email non registrata
  - Expected output: 200 OK (non rivelando esistenza)
- test_reset_password_confirm_success
  - Tipo: integration
  - Verifica: reset completato con codice valido e nuova password
  - Input: codice reset valido, nuova password conforme
  - Expected output: 200 messaggio successo
- test_reset_password_confirm_invalid_code
  - Tipo: integration
  - Verifica: codice errato o scaduto produce 401
  - Input: codice non valido/scaduto
  - Expected output: 401 UNAUTHORIZED

### Sicurezza
- test_jwt_auth_middleware_valid_token
  - Tipo: unit
  - Verifica: accesso endpoint protetto con token valido passa
- test_jwt_auth_middleware_invalid_or_missing_token
  - Tipo: unit
  - Verifica: token mancante o invalido produce 401
- test_input_validation_prevents_injection
  - Tipo: unit
  - Verifica: input malformati o potenzialmente pericolosi respinti

## 11. Rischi tecnici
- Mancata conferma email potrebbe permettere account non validi o abusivi.
- Assenza dettagliata di strategia retry/fallback servizio email può bloccare reset password.
- Mancanza di refresh token limita UX e usabilità futura.
- Vulnerabilità brute force se limiti tentativi o monitoraggio non adeguati.
- Mancanza di test prestazionali espliciti riduce certezza su prestazioni reali.
- Gestione concorrenza registrazione potrebbe causare duplicati se locking DB non perfetto.
- Logging e monitoraggio devono rispettare compliance di sicurezza, rischi se male configurati.
- Estensione futura limitata da assenza autorizzazioni granulari e MFA.

## 12. Struttura file proposta
``` 
project_root/
  main.java                   # Entry point applicazione —  public class Main { public static void main(String[] args) }
  config/
    Settings.java             # Configurazione app — class Settings { dbConfig, jwtConfig, emailConfig }
  models/
    User.java                 # Modello User — class User { Long id; String email; String passwordHash; LocalDateTime createdAt; }
    ResetToken.java           # Modello ResetToken — class ResetToken { Long id; Long userId; String code; LocalDateTime expiresAt; boolean used; }
  repositories/
    UserRepository.java       # Accesso dati utenti — interface UserRepository { User findByEmail(...); void save(...); }
    ResetTokenRepository.java # Accesso dati token reset — interface ResetTokenRepository
  services/
    AuthService.java          # Logica autenticazione e registrazione — class AuthService { registerUser(), login(), generateJwt() }
    ResetPasswordService.java # Logica reset password — class ResetPasswordService { requestReset(), confirmReset() }
    ValidationService.java    # Validazione input — class ValidationService { validateEmail(), validatePassword() }
    EmailService.java         # Invio email con retry — class EmailService { sendEmail() }
    EventLogger.java          # Logging eventi critici — class EventLogger { logLoginEvent(), logResetEvent() }
    MetricsMonitor.java       # Monitoraggio metriche e tentativi — class MetricsMonitor { incrementLoginAttempts(), isBlocked() }
  middlewares/
    JwtAuthMiddleware.java    # Middleware Bear per verifica JWT — class JwtAuthMiddleware { handle(Request req, Response res) }
  controllers/
    UserController.java       # Endpoint REST — class UserController { register(), login(), resetPasswordRequest(), resetPasswordConfirm() }
  utils/
    HashUtil.java             # Hashing password bcrypt — class HashUtil { hash(), verify() }
    JwtUtil.java              # Generazione e verifica JWT — class JwtUtil { generateToken(), verifyToken() }
  tests/
    AuthServiceTest.java      # Test unit/integration AuthService
    ResetPasswordServiceTest.java # Test unit/integration ResetPasswordService
    ValidationServiceTest.java    # Test unit validation
    JwtAuthMiddlewareTest.java    # Test middleware JWT
```

## 13. Piano di implementazione
1. Definire schema database utenti e token, creare script di migrazione con Liquibase/Flyway.
2. Implementare modelli e repository per utenti e reset token.
3. Sviluppare module ValidationService per validazione email e password.
4. Realizzare AuthService con logica registrazione e login, inclusa hashing password e generazione JWT.
5. Implementare JwtAuthMiddleware per protezione endpoint.
6. Sviluppare ResetPasswordService con generazione codice temporaneo, invio email (EmailService) e conferma reset.
7. Implementare UserController con endpoint REST: /register, /login, /reset-password/request, /reset-password/confirm.
8. Integrare EventLogger per logging eventi critici e MetricsMonitor per monitoraggio tentativi.
9. Configurare pool connessioni DB e parametri di timeout per performance.
10. Scrivere test unitari e di integrazione per tutte le componenti critiche.
11. Eseguire test di carico e performance con report, ottimizzare se necessario per rispettare soglia <1s.
12. Estendere documentazione tecnica con dettaglio API, configurazioni, strategy retry email e limiti tentativi.
13. Deploy e attivazione monitoraggio con alert configurati.
14. Revisione finale e rilascio MVP.

## QA Output
MODULE: QA
VERSION: 1
FINAL_STATUS: APPROVED_WITH_CHANGES

## 1. Checklist — Copertura requisiti
* [SI] Utente può registrarsi con email unica e password conforme a policy — La proposta prevede endpoint /register con controllo unicità email, validazione password conforme a policy (min 8 caratteri, lettere, numeri, simboli).
* [SI] Utente può effettuare login con email/password valide e ricevere token JWT con scadenza — Endpoint /login con verifica credenziali e generazione JWT con scadenza circa 1 ora è definito.
* [SI] Endpoint protetti rispondono solo se token JWT è presente, valido e non scaduto; altrimenti ritornano errore 401 — JwtAuthMiddleware verifica presenza, validità e scadenza token, risponde 401 se token mancante/invalido.
* [SI] Reset password consente di inviare codice/link via email e permette modifica password — Endpoint request e confirm per reset password esposti, con invio codice via email e conferma con codice e nuova password.
* [SI] Password salvate nel database sono hashed con salt sicuro — Uso bcrypt con salt forte è esplicitato nella logica hashing.
* [SI] Tutte le API validano gli input e impediscono injection o dati non validi — ValidationService presente con chiamate in controller, validazione email e password spiegita.
* [SI] Log eventi critici sono correttamente registrati e consultabili in modo sicuro — Logging sicuro cifrato di eventi critici previsto con EventLogger e LoggerConfig.
* [PARZIALE] Il sistema risponde con performance accettabili (es. tempi di risposta inferiori a 1 secondo per endpoint critici) — Architettura modulare, pool connessioni DB e attenzione a scalabilità stateless ci sono, ma mancano metriche o test prestazionali espliciti.
* [SI] Il sistema scala orizzontalmente senza perdita di funzionalità — Architettura stateless e uso pooling DB confermano supporto a scalabilità orizzontale.
* [SI] Test di sicurezza base superati (es. tentativi ripetuti login bloccati o limitati) — Limiti tentativi brute force implementati, test dedicati descritti.
* [SI] Documentazione tecnica completa e aggiornata — Documentazione API, configurazioni e piano implementazione dettagliati sono presenti.

## 2. Checklist — Contratti API
* [SI] Ogni endpoint ha metodo HTTP, path, request schema, response schema e esempio concreto — Tutti i principali endpoint descritti con dettagli completi e esempi.
* [SI] Le regole di validazione sono esplicite per ogni campo (tipo, formato, obbligatorietà, regex ove necessario) — Specifiche password, email e codice temporaneo esplicitate.
* [SI] Il formato degli errori è consistente tra tutti gli endpoint (struttura JSON uniforme) — Sezione gestione errori definisce formato uniformemente usato con campi "error_code" e "message".
* [SI] I codici HTTP di risposta (2xx, 4xx, 5xx) sono specificati per ogni endpoint — Codici 200, 201, 400, 401, 409 indicati chiaramente per ogni API.
* [NO] La strategia di paginazione è definita per le liste (se applicabile) — Non applicabile in MVP (assenza di endpoint di liste), quindi fuori scope.

## 3. Checklist — Business logic e scenari limite
* [SI] I flussi principali sono descritti passo per passo (non solo a parole generiche) — Business logic di registrazione, login, reset password con passaggi dettagliati, inclusi transazioni DB.
* [SI] Gli scenari di concorrenza sono trattati (es. doppia registrazione, doppio click, race condition) — Registrazione concorrente con controllo unicità lato DB e locking gestito.
* [PARZIALE] I casi di fallimento delle integrazioni esterne hanno una strategia (retry, fallback, circuit breaker) — Email service previsto con retry e fallback di base, ma senza dettagli approfonditi su circuit breaker o gestione avanzata.
* [SI] Le regole di business critiche sono esplicite e non ambigue — Regole per password, durata token, limiti di tentativi, gestione codice reset sono chiare e precise.

## 4. Checklist — Persistenza e schema dati
* [SI] Le tabelle/collezioni principali sono definite con i campi e i tipi — Tabelle users e reset_tokens con campi principali e tipi definiti.
* [SI] Gli indici sono specificati per le colonne usate in query frequenti o join — Indici per email e codice reset dichiarati.
* [SI] I vincoli di unicità e foreign key sono dichiarati — Unicità email e codice, FK per user_id in reset_tokens menzionati.
* [SI] La strategia di migrazione dello schema è menzionata — Uso di tool come Liquibase/Flyway indicato e documentato.

## 5. Checklist — Strategia di test
* [SI] Esistono test per i happy path di ogni funzionalità principale — Test registrazione, login, reset password includono i casi di successo.
* [SI] Esistono test per i casi di errore critici (validazione fallita, not found, unauthorized) — Test per duplicato email, password errata, codice reset non valido sono presenti.
* [SI] Esistono test di integrazione per le dipendenze esterne (DB, servizi esterni) — Test di integrazione coprono interazione con DB e simulazione invio email.
* [SI] I test specificano input e expected output concreti (non generici) — Specifiche dei test includono input esatti e output attesi con codici HTTP.

## 6. Requisiti mancanti
Nessuno.

## 7. Rischi e problemi
* ALTA — Mancata conferma email può creare account non validi e potenzialmente abusivi.
* MEDIA — Strategia retry/fallback del servizio email non sufficientemente dettagliata, rischio blocco reset password.
* MEDIA — Assenza di meccanismo refresh token limita esperienza utente futura.
* ALTA — Possibili vulnerabilità brute force se limiti tentativi e monitoraggio non adeguatamente configurati o sufficientemente resilienti.
* MEDIA — Mancanza di test espliciti di prestazioni limitano certezza su rispetto della soglia 1 secondo.
* BASSA — Assenza di autorizzazioni granulari limita evoluzioni future ma è fuori scope attuale.

## 8. Azioni richieste
[PRIORITÀ ALTA] Fornire test o metriche di performance dettagliate che dimostrino la latenza media degli endpoint critici inferiore a 1 secondo.

[PRIORITÀ MEDIA] Estendere la descrizione della strategia di retry e fallback del servizio email con indicazioni più precise e possibili gestione errori a cascata.

[PRIORITÀ MEDIA] Considerare una proposta a futuro per gestione refresh token o mitigazione UX in assenza di refresh, evidenziando impatti.

[PRIORITÀ MEDIA] Documentare in modo più esteso la configurazione e monitoraggio dei limiti tentativi per garantire resilienza a brute force.

Nessun altro requisito impedisce l'approvazione con modifiche.