MODULE: BACKEND VERSION: 1
## 1. Obiettivo backend
Realizzare un backend Java basato su REST API e framework Bear per gestire l’autenticazione utenti tramite email e password. Il backend deve supportare la registrazione con validazione email unica e policy password, login con generazione di token JWT firmati e con scadenza, protezione di endpoint tramite middleware JWT, e funzionalità base di reset password con invio codice/link temporaneo via email. Il sistema deve garantire sicurezza (password hashed con salt, validazione input, logging eventi critici), performance adeguate (<1s risposta per endpoint critici) e scalabilità orizzontale usando PostgreSQL come database. L’architettura deve permettere facile monitoraggio e gestione degli errori coerente.

## 2. Assunzioni tecniche
- Validazione email con controllo unicità senza conferma tramite link.
- Password conformi a policy standard: minimo 8 caratteri, con lettere, numeri e simboli.
- Token JWT validi per circa un’ora, firmati con chiave segreta, senza refresh token MVP.
- Reset password base tramite invio link/codice via email; validità temporale limitata.
- Limiti di tentativi login/reset per mitigazione brute force implementati come meccanismo base.
- Persistenza utenti e token salvata in PostgreSQL, con gestione concorrente sicura.
- Logging sicuro di eventi critici e monitoraggio base per rilevamento anomalie.
- Backend esclusivamente API REST, nessun frontend o UI.
- Autenticazione senza MFA o social login.
- Nessuna gestione ruoli o autorizzazioni granulari per MVP.

## 3. Architettura backend
Architettura a 3 livelli:
- Presentation Layer: REST API implementate con framework Bear, espongono endpoint per registrazione, login, reset password e endpoint protetti tramite middleware.
- Business Logic Layer: moduli per validazione input, gestione utenti, generazione/verifica JWT, sicurezza password e log degli eventi.
- Data Layer: modulo di accesso a PostgreSQL tramite DAO/Repository con schema utenti e token, transazioni per coerenza dati e gestione concorrenza in registrazione e sessioni.

Il middleware Bear intercetta richieste a endpoint protetti, verifica validità e scadenza token JWT, rigettando richieste non autorizzate con 401. La scalabilità è favorita da statelessness tramite JWT e database PostgreSQL configurato per supporto a carichi crescenti.

## 4. Moduli e responsabilità
- UserModule: gestione registrazione utenti, hashing password con salt, validazione e verifica unicità email.
- AuthModule: gestione login, verifica credenziali, generazione token JWT firmati e validazione token.
- ResetPasswordModule: gestione richiesta reset, generazione codice/link temporanei, invio email e modifica password.
- JwtMiddleware: filtro Bear per protezione API via verifica token JWT valido e non scaduto.
- DatabaseModule: DAO per accesso dati utenti e token, gestione transazionale.
- LoggingModule: logging sicuro eventi critici (login, reset, errori), livello di log configurabile.
- ValidationModule: validazione rigorosa input API per prevenzione injection e dati non validi.
- MonitoringModule: monitoraggio base e alert per tentativi violazione, errori di sistema.

## 5. API principali

### POST /api/v1/auth/register
- Request body:
  ```json
  {
    "email": "string, email valida, unica",
    "password": "string, min 8 caratteri, lettere, numeri, simboli"
  }
  ```
- Response body:
  - 201 Created
  ```json
  {
    "message": "User registered successfully."
  }
  ```
  - 400 Bad Request (validation error)
  ```json
  {
    "error": "Detailed validation message."
  }
  ```
  - 409 Conflict (email già esistente)
  ```json
  {
    "error": "Email already registered."
  }
  ```
- Esempio request:
  ```json
  {
    "email": "user@example.com",
    "password": "P@ssw0rd!"
  }
  ```
- Esempio response:
  ```json
  {
    "message": "User registered successfully."
  }
  ```

### POST /api/v1/auth/login
- Request body:
  ```json
  {
    "email": "string",
    "password": "string"
  }
  ```
- Response body:
  - 200 OK
  ```json
  {
    "token": "jwt_token_string",
    "expires_in": 3600
  }
  ```
  - 401 Unauthorized (credenziali errate)
  ```json
  {
    "error": "Invalid email or password."
  }
  ```
- Esempio request:
  ```json
  {
    "email": "user@example.com",
    "password": "P@ssw0rd!"
  }
  ```
- Esempio response:
  ```json
  {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expires_in": 3600
  }
  ```

### POST /api/v1/auth/reset-password/request
- Request body:
  ```json
  {
    "email": "string"
  }
  ```
- Response body:
  - 200 OK
  ```json
  {
    "message": "Reset link/code sent to email."
  }
  ```
  - 404 Not Found
  ```json
  {
    "error": "Email not registered."
  }
  ```
- Esempio request:
  ```json
  {
    "email": "user@example.com"
  }
  ```
- Esempio response:
  ```json
  {
    "message": "Reset link/code sent to email."
  }
  ```

### POST /api/v1/auth/reset-password/confirm
- Request body:
  ```json
  {
    "email": "string",
    "reset_code": "string",
    "new_password": "string"
  }
  ```
- Response body:
  - 200 OK
  ```json
  {
    "message": "Password successfully reset."
  }
  ```
  - 400 Bad Request (reset_code non valido o scaduto, password non valida)
  ```json
  {
    "error": "Invalid or expired reset code."
  }
  ```
- Esempio request:
  ```json
  {
    "email": "user@example.com",
    "reset_code": "123456",
    "new_password": "N3wP@ssw0rd!"
  }
  ```
- Esempio response:
  ```json
  {
    "message": "Password successfully reset."
  }
  ```

### Endpoint protetti esempio

GET /api/v1/user/profile
- Protetto da middleware JWT
- Request header:
  ```
  Authorization: Bearer <jwt_token>
  ```
- Response body:
  - 200 OK
  ```json
  {
    "email": "user@example.com",
    "created_at": "2024-06-01T12:00:00Z"
  }
  ```
  - 401 Unauthorized (token mancante, invalido o scaduto)
  ```json
  {
    "error": "Unauthorized: token missing or invalid."
  }
  ```

## 6. Business logic
- Registrazione: validazione email e password; controllo unicità email direttamente a livello DB con lock di concorrenza; password hashata con algoritmo salato (es. bcrypt) prima persistenza.
- Login: verifica password confrontando hash; in caso positivo generazione token JWT firmato con payload email, issuedAt, expiration; logging evento login.
- Reset password: generazione codice temporaneo univoco memorizzato con scadenza; invio tramite email (mock o SMTP realistica); verifica codice durante conferma; aggiornamento password con hashing; invalidamento codici usati.
- Middleware JWT: estrazione token da header, decoding e verifica firma/scadenza; rifiuto se token non valido; log tentativi non autorizzati.
- Limitazione tentativi: contatore tentativi falliti login e reset per IP e/o utente con blocco temporaneo.
- Validazioni input per impedire injection SQL, scripting o payload invalidi.

## 7. Persistenza e integrazioni
- PostgreSQL:
  - Tabella users: id (UUID PK), email (varchar unique), password_hash (varchar), salt (varchar), created_at (timestamp), updated_at (timestamp).
  - Tabella reset_password_tokens: id (UUID PK), user_id (FK), token_code (varchar), expires_at (timestamp), used (bool).
  - Indici su email in users, user_id in reset tokens.
  - Vincoli di unicità su email e token_code.
- Integrazione SMTP/servizio email per invio reset link/codice.
- Configurazione connessione DB con pool dimensionato per scalabilità.
- Transazioni per operazioni critiche (es. registrazione, reset password).

## 8. Autenticazione e autorizzazione
- Autenticazione tramite email+password con hash e salt.
- JWT come token di autenticazione, firmato con HS256 o altro algoritmo sicuro, emesso con scadenza 1 ora.
- Nessun refresh token per MVP.
- Middleware Bear che valida token su tutte API protette, risponde 401 su token assente/errato/scaduto.
- Nessuna gestione ruoli o permessi granulari.

## 9. Gestione errori
- Risposte API coerenti con JSON in formato {"error": "messaggio"} per errori client 4xx e server 5xx.
- Codici HTTP rigorosi: 400 per input non valido, 401 per autenticazione fallita, 404 per risorse non trovate, 409 per conflitti (email duplicata).
- Log esteso per errori backend su file sicuro e/o sistema di logging centralizzato con livelli di gravità.
- Gestione eccezioni globali per intercettare errori imprevisti e restituire messaggi generici 500 con log dettagliati.
- Validazione input preventiva non permette payload non conformi.

## 10. Strategia di test backend
- test_register_user_success (unit): verifica registrazione utente con email valida e password conforme; input: email+password validi; expected output: 201 Created.
- test_register_user_duplicate_email (unit): verifica gestione email duplicata; input: email già registrata; expected output: 409 Conflict.
- test_login_success (unit): verifica login con credenziali corrette; output JWT token e 200.
- test_login_invalid_password (unit): verifica risposta 401 con password errata.
- test_jwt_middleware_accept_valid_token (integration): verifica accesso a endpoint protetto con token valido.
- test_jwt_middleware_reject_invalid_token (integration): verifica 401 con token non valido/scaduto.
- test_reset_password_request_existing_email (unit): verifica invio codice/reset link email.
- test_reset_password_request_unknown_email (unit): verifica 404 per email assente.
- test_reset_password_confirm_success (unit): verifica reset password con codice valido.
- test_reset_password_confirm_invalid_code (unit): verifica 400 con codice errato o scaduto.
- test_input_validation_reject_bad_email (unit): input invalidi su email, risposta 400.
- test_rate_limiting_login (integration): verifica blocchi tentativi e limitazioni per brute force.
- test_performance_endpoints (e2e): verifica tempi risposta <1s su register, login, reset.
- test_logging_events (integration): verifica scrittura corretta log in casi critici.

## 11. Rischi tecnici
- Gestione concorrenza registrazione simultanea con stessa email deve prevenire duplicati tramite vincoli DB e logica retry.
- Mancanza refresh token potrebbe limitare usabilità; futura estensione necessario.
- Potenziali attacchi brute force se limiti e monitoraggio insufficienti.
- Implementazione robusta della validazione input essenziale per prevenire injection.
- Dipendenza da sistema email per reset password; problemi delivery potrebbero influire UX.
- Logging e monitoraggio devono essere configurati per rispettare compliance sicurezza.
- Scalabilità orizzontale DBA e backend da testare sotto carico.

## 12. Struttura file proposta
``` 
project_backend/
  Application.java                     # Entry point – public static void main(String[] args)
  config/
    Settings.java                     # Configurazione app – class Settings {jwtSecret, dbConfig, emailConfig}
  models/
    User.java                        # Modello User – class User {UUID id, String email, String passwordHash, String salt, Timestamp createdAt}
    ResetPasswordToken.java          # Modello reset token – class ResetPasswordToken {UUID id, UUID userId, String tokenCode, Timestamp expiresAt, boolean used}
  repositories/
    UserRepository.java              # DAO utenti – findByEmail(), save(), existsByEmail()
    ResetPasswordTokenRepository.java # DAO reset token – save(), findByTokenCode(), markUsed()
  services/
    UserService.java                 # Logica utenti – registerUser(), hashPassword(), validatePassword(), checkEmailUniqueness()
    AuthService.java                 # Login e token – authenticate(), generateJWT(), verifyPassword()
    ResetPasswordService.java        # Reset pwd – createResetToken(), sendResetEmail(), verifyResetCode(), updatePassword()
  middleware/
    JwtAuthenticationFilter.java    # Bear middleware – doFilter() verifica token JWT, gestione 401
  controllers/
    AuthController.java             # REST endpoints auth – register(), login(), resetPasswordRequest(), resetPasswordConfirm()
    UserController.java             # Endpoint protetti – profile()
  utils/
    PasswordUtils.java              # Hashing, salt generation – hashPassword(), generateSalt(), verifyPassword()
    JwtUtils.java                   # JWT utils – createToken(), validateToken()
    ValidationUtils.java            # Validazioni input – emailFormat(), passwordPolicy(), inputSanitization()
    EmailSender.java                # Invio email – sendEmail()
  logging/
    LoggerConfig.java               # Configurazione logging – livelli, formati
  monitoring/
    MonitorService.java             # Monitoraggio base – alert tentativi sospetti, errori critici
  tests/
    UserServiceTest.java            # Unit test UserService
    AuthServiceTest.java            # Unit test AuthService
    ResetPasswordServiceTest.java   # Unit test ResetPasswordService
    JwtAuthenticationFilterTest.java # Integration test middleware
    AuthControllerTest.java         # Integration test endpoint auth
    PerformanceTests.java           # E2E test prestazioni endpoint
```

## 13. Piano di implementazione
1. Progettare schema PostgreSQL utenti e token con vincoli, indici, migrator.
2. Implementare UserService con hashing password, validazione e controllo unicità email.
3. Implementare endpoint /register in AuthController con validazione input e gestione errori.
4. Implementare AuthService per autenticazione, verifica password e generazione JWT.
5. Implementare endpoint /login con risposta token e gestione errori.
6. Sviluppare JwtAuthenticationFilter come middleware Bear per protezione endpoint.
7. Implementare ResetPasswordService con generazione codice, invio email e conferma reset.
8. Implementare endpoint reset password request e confirm.
9. Integrare logging sicuro con LoggerConfig e chiamate logging eventi critici.
10. Inserire validazioni input rigorose in ValidationUtils e applicarle in controller/service.
11. Realizzare test unitari su servizi e integrazione middleware e controller.
12. Effettuare test di performance e sicurezza incluse limitazioni tentativi.
13. Configurare monitoraggio base con alert su eventi critici.
14. Documentare API con esempi request/response, codici HTTP, validazioni, gestione errori.
15. Revisionare gestione concorrenza, politiche di sicurezza password e limiti tentativi.
16. Preparare ambiente deploy scalabile orizzontalmente e testare carico.
17. Rilascio MVP con monitoraggio e logging attivi, pronto per estensioni future.