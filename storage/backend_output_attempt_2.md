MODULE: BACKEND VERSION: 1
## 1. Obiettivo backend
Sviluppare un backend Java basato su REST API che gestisca l’autenticazione utenti stateless utilizzando JWT, consentendo registrazione tramite email e password, login con creazione di token JWT firmati, protezione di endpoint tramite middleware Bear, e reset password di base con invio link temporaneo via email. Il backend dovrà garantire sicurezza dei dati sensibili (password hashed e salted), logging centralizzato conforme GDPR, validazione input robusta per prevenzione injection, e scalabilità orizzontale senza gestione complessa di ruoli o meccanismi avanzati.

## 2. Assunzioni tecniche
- Backend Java conforme a framework Bear, implementato come API REST stateless.
- Database PostgreSQL per persistenza utenti con vincoli univoci su email.
- JWT per autenticazione stateless, con token firmati e scadenza fissa (es. 1 ora).
- Comunicazione HTTPS garantita via configurazione ambiente esterna.
- Password salvate con algoritmo hashing sicuro (es. bcrypt) con salt.
- Endpoint pubblici: registrazione, login, reset password; tutti gli altri protetti da validazione JWT.
- Reset password base implementato tramite link temporalmente limitato inviato via email.
- Logging centralizzato anonimizzato per eventi autenticazione.
- Nessuna gestione ruoli, social login o meccanismi avanzati di reset password.
- Gestione errori generici e messaggi di sicurezza uniformi.
- Strategia SMTP per invio email con gestione errori via retry/fallback implementata.
- Gestione migrazione schema DB pianificata tramite tool esterno (es. Flyway).
- Concorrenza gestita almeno con lock DB atomico per validazione email unica in registrazione.

## 3. Architettura backend
Architettura modulare e stratificata:
- Controller/Router REST Bear per esposizione API.
- Service Layer per logica business (AuthService, UserService, JwtService, EmailService, LoggingService).
- Repository Layer per accesso PostgreSQL tramite DAO/ORM.
- Middleware Bear per verifica token JWT su endpoint protetti.
- Configurazione sicura esterna per parametri JWT, SMTP, DB, logging.
- Statelesness garantito tramite JWT senza gestione sessioni server-side.
- Logging centralizzato su sistema esterno garantendo compliance GDPR.
- Validazione input centralizzata tramite componenti di validazione.
- Gestione errori uniforme in livello controller per standardizzazione risposte.

## 4. Moduli e responsabilità
- User Module: gestione modello utenti, validazione registro.
- Auth Module: login, verifica credenziali, generazione JWT.
- Password Module: hashing/salting password, reset password e generazione link.
- JWT Module: creazione, validazione e configurazione token.
- Middleware JWT: verifica token per accesso endpoint protetti.
- Email Module: invio mail reset password con retry/fallback.
- Logging Module: registrazione centralizzata eventi autenticazione con anonimizzazione.
- Validation Module: validazione sicurezza input (email, password).
- Config Module: lettura configurazioni sicure ambientali.
- DB Access Module: gestione accesso e migrazione schemi DB.
  
## 5. API principali

### POST /api/v1/register
- Request body schema:
  ```json
  {
    "email": "string (email RFC valida, obbligatorio)",
    "password": "string (min 8 caratteri, obbligatorio)"
  }
  ```
- Response:
  - 201 Created: `{ "message": "User registered successfully" }`
  - 400 Bad Request: `{ "error": "Invalid email or password format" }`
  - 409 Conflict: `{ "error": "Email already registered" }`
- Esempio request:
  ```json
  {
    "email": "user@example.com",
    "password": "securePassword123"
  }
  ```
- Esempio response:
  ```json
  {
    "message": "User registered successfully"
  }
  ```

### POST /api/v1/login
- Request body schema:
  ```json
  {
    "email": "string (email valida, obbligatorio)",
    "password": "string (obbligatorio)"
  }
  ```
- Response:
  - 200 OK:
    ```json
    {
      "token": "jwt_token_string",
      "expires_in": 3600
    }
    ```
  - 400 Bad Request: `{ "error": "Invalid request format" }`
  - 401 Unauthorized: `{ "error": "Invalid credentials" }`
- Esempio request:
  ```json
  {
    "email": "user@example.com",
    "password": "securePassword123"
  }
  ```
- Esempio response:
  ```json
  {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expires_in": 3600
  }
  ```

### POST /api/v1/reset-password
- Request body schema:
  ```json
  {
    "email": "string (email valida, obbligatorio)"
  }
  ```
- Response:
  - 200 OK: `{ "message": "If the email is registered, a reset link has been sent" }`
  - 400 Bad Request: `{ "error": "Invalid email format" }`
- Esempio request:
  ```json
  {
    "email": "user@example.com"
  }
  ```
- Esempio response:
  ```json
  {
    "message": "If the email is registered, a reset link has been sent"
  }
  ```

### GET /api/v1/protected-resource (esempio endpoint protetto)
- Headers: `Authorization: Bearer <jwt_token>`
- Response:
  - 200 OK: `{ "data": "protected content" }`
  - 401 Unauthorized: `{ "error": "Authentication required" }`
- Esempio request header:
  ```
  Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
  ```
- Esempio response:
  ```json
  {
    "data": "protected content"
  }
  ```

## 6. Business logic
- Registrazione: valida formato email/password, controlla unicità email con lock DB per evitare duplicati, esegue hashing e salt password, salva utente in DB.
- Login: ricerca utente per email, verifica hash password, genera token JWT firmato con payload minimo (userId, email, exp).
- Middleware JWT: intercetta chiamate endpoint protetti, valida firma e scadenza token, rifiuta 401 se non valido.
- Reset password: riceve email, genera token reset temporaneo, salva in DB con scadenza breve, invia link via email con retry in caso di errore, risposta all’utente sempre generica per sicurezza.
- Logging: registra eventi login, registrazione, reset e fallimenti senza dati sensibili riconducibili all’utente.
- Validazioni: email RFC valida, password con almeno 8 caratteri, input sanificati per evitare injection.
- Scadenza token JWT configurata centralmente (default 1 ora).
- Architettura stateless per consentire scalabilità senza gestione sessione server.

## 7. Persistenza e integrazioni
- PostgreSQL con tabella utenti:
  - id (UUID PK)
  - email (varchar unique, indexed)
  - password_hash (varchar)
  - salt (varchar)
  - created_at (timestamp)
  - updated_at (timestamp)
- Tabella reset_password_tokens:
  - token (UUID/Pk)
  - user_id (FK utenti)
  - expires_at (timestamp)
- Integrazione con server SMTP configurabile per invio email reset con retry/fallback.
- Migrazione schema DB gestita tramite tool esterno (es. Flyway), mantenendo versioning e rollback.
- Logging centralizzato via client esterno o middleware con anonimizzazione e crittografia.

## 8. Autenticazione e autorizzazione
- Autenticazione basata su JWT firmati simmetricamente tramite chiave segreta sicura.
- Token ha payload minimo e scadenza fissa configurabile.
- Middleware Bear intercetta e valida token per consentire accesso a endpoint protetti esclusi registrazione/login/reset.
- Nessuna autorizzazione differenziata per ruoli o permessi (out of scope).
- Nessuna gestione revoca token (token scadono naturalmente).
- Password protette hashing sicuro + salt univoco per utente.

## 9. Gestione errori
- Risposte uniformi in JSON con campo chiave "error" per errori cliente (4xx).
- Messaggi di errore generici su reset password per non rivelare esistenza email.
- 409 Conflict per email duplicate alla registrazione.
- 401 Unauthorized per token mancanti o invalidi.
- 400 Bad Request per formati request non validi.
- Logging di errori con anonimizzazione per analisi e monitoraggio.
- Gestione errori integrazione SMTP con retry (configurabile) e fallback log alert.
- Errore DB o infrastruttura restituisce 500 con messaggio generico "Internal server error".

## 10. Strategia di test backend

### Test modulo registrazione
- test_register_user_success (integration)
  - Verifica registrazione con email e password valide.
  - Input: email valida univoca + password 8+ caratteri.
  - Output: 201 Created, messaggio successo.

- test_register_duplicate_email (integration)
  - Verifica errore 409 per email già esistente.
  - Input: email già registrata + password.
  - Output: 409 Conflict.

- test_register_invalid_email_format (unit)
  - Verifica validazione email non valida.
  - Input: email non conforme RFC.
  - Output: 400 Bad Request.

### Test modulo login
- test_login_success (integration)
  - Login corretto con generazione token JWT.
  - Input: email e password corrette.
  - Output: 200 OK + JWT token.

- test_login_invalid_password (integration)
  - Input credenziali errate.
  - Output: 401 Unauthorized.

- test_login_missing_fields (unit)
  - Mancanza campi obbligatori.
  - Output: 400 Bad Request.

### Test modulo reset password
- test_reset_password_email_sent (integration)
  - Invio email reset link con successo.
  - Input: email esistente.
  - Output: 200 OK messaggio generico.

- test_reset_password_email_not_registered (integration)
  - Invio con email non registrata.
  - Output: 200 OK (stesso messaggio per sicurezza).

- test_reset_password_invalid_email_format (unit)
  - Email formattata male.
  - Output: 400 Bad Request.

### Test middleware JWT
- test_access_protected_with_valid_token (integration)
  - Accesso endpoint protetto con token valido.
  - Output: 200 OK.

- test_access_protected_with_invalid_token (integration)
  - Accesso con token scaduto o modificato.
  - Output: 401 Unauthorized.

- test_access_protected_without_token (integration)
  - Accesso senza token.
  - Output: 401 Unauthorized.

### Test integrazione SMTP
- test_email_send_success (integration)
  - Invio email via SMTP mock positivo.
- test_email_send_retry_on_failure (integration)
  - Retry in caso di errore SMTP.
  
## 11. Rischi tecnici
- Assenza meccanismi anti-brute force può esporre login ad attacchi forza bruta.
- Mancata revoca token JWT impedisce logout remoto o invalidamento token compromessi.
- Mancanza esplicita gestione concurrency avanzata oltre lock DB univocità può portare a race condition rare in registrazione.
- Assenza strategia fallback robusta per SMTP può causare mancato invio reset password.
- Mancanza piano migrazione DB esplicito rischia downtime o incoerenze su upgrade schema.
- Sincronizzazione oraria distribuita deve essere monitorata per validità token JWT.
- Esclusione conferma email può permettere registrazioni con email errate.
- Privacy logging e GDPR richiedono attenzione nell’implementazione.

## 12. Struttura file proposta
``` 
project_backend/
  main.java              # Entry point applicazione — public static void main(String[] args)
  config/
    Settings.java        # Configurazione app — class Settings { getJwtSecret(), getSmtpServer() }
  models/
    User.java            # Modello User — class User { UUID id, String email, String passwordHash, String salt, timestamps }
    ResetToken.java      # Modello Reset Password Token — class ResetToken { UUID token,UUID userId, Instant expiresAt }
  repositories/
    UserRepository.java  # Accesso DB utenti — interface UserRepository { findByEmail(), save(), ...)
    ResetTokenRepository.java # Accesso DB token reset
  services/
    AuthService.java     # Logica autenticazione — boolean validatePassword(), String generateJwtToken(User)
    UserService.java     # Gestione utenti — User registerUser(), ...
    PasswordService.java # Hashing/salting password — String hashPassword(String plain), boolean verifyPassword()
    EmailService.java    # Invio email — void sendResetPasswordEmail(String email, String link)
    JwtService.java      # Gestione token JWT — String createToken(), boolean validateToken()
    LoggingService.java  # Logging centralizzato — void logEvent(String eventType, Map<String,Object> data)
    ValidationService.java # Validazioni input — boolean isValidEmail(), boolean isStrongPassword()
  middleware/
    JwtAuthenticationFilter.java # Middleware verificatore token JWT — void doFilter()
  routers/
    AuthController.java  # Endpoint auth — register(), login(), resetPassword()
    ProtectedController.java # Endpoint protetti — esempio getProtectedData()
  utils/
    DateTimeUtils.java   # Utility data ora, formati, scadenze
    RetryUtils.java      # Gestione retry per SMTP e altre operazioni
  tests/
    UserServiceTest.java      # Unit test UserService
    AuthServiceTest.java      # Unit test AuthService
    PasswordServiceTest.java  # Unit test PasswordService
    EmailServiceIntegrationTest.java # Test integrazione SMTP mock
    AuthControllerIntegrationTest.java # Test e2e autenticazione
    JwtMiddlewareTest.java    # Test middleware JWT
```

## 13. Piano di implementazione
1. Definizione schema utenti e tabelle reset password in PostgreSQL, configurazione migrazione DB.
2. Implementazione modulo password con hashing e salting sicuro.
3. Sviluppo UserService con logica registrazione e verifica unicità email con lock DB.
4. Creazione AuthService per login e generazione JWT con configurazione durata.
5. Implementazione middleware Bear per verifica token JWT su tutti gli endpoint protetti.
6. Sviluppo AuthController con endpoint register, login e reset password; integrazione EmailService e RetryUtils.
7. Configurazione LoggingService centralizzato per eventi autenticazione senza dati sensibili.
8. Validazione input consolidata con ValidationService per prevenzione injection e formati corretti.
9. Configurazione ambiente HTTPS e parametri sicurezza (JWT secret, SMTP, DB).
10. Scrittura test unitari e di integrazione per tutti i casi di successo e errori.
11. Esecuzione test con mock SMTP e DB, verifica compliance GDPR e sicurezza.
12. Documentazione API con esempi request/response, gestione errori e decoratori middleware.
13. Deployment e monitoraggio log eventi, validazione scalabilità orizzontale e sincronizzazione oraria.
14. Pianificazione future estensioni: gestione retry avanzata SMTP, migrazione schema evolutiva, valutazione anti-brute force e revoca token.