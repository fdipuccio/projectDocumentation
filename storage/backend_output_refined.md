MODULE: BACKEND VERSION: 1
## 1. Obiettivo backend
Realizzare un backend REST API per la gestione dell’autenticazione utenti che consenta registrazione con email e password, login con generazione di token JWT per sessioni stateless, protezione di endpoint via validazione token JWT, e gestione del reset password tramite invio email con link/codice temporaneo. Il sistema deve garantire sicurezza robusta, scalabilità orizzontale, e monitoraggio minimo operativo.

## 2. Assunzioni tecniche
- Linguaggio Java con framework per sviluppo REST (es. Spring Boot).
- Database relazionale PostgreSQL per storage utenti, password hashate, e token/reset code temporanei.
- Gestione token JWT con secret key robusta, firma HMAC e scadenza configurabile (~1 ora).
- Hashing password con algoritmo sicuro (preferibilmente Argon2 o bcrypt).
- Middleware/filtro per autenticazione basata su token JWT.
- Integrazione di Bear da definire e chiarire come framework/libreria ausiliaria per sicurezza e logging.
- Meccanismi di mitigazione attacchi brute force e parziale rate limiting da parametrizzare.
- Invio email per reset password con sistema esterno configurabile.
- Sistema stateless progettato per containerizzazione e scalabilità orizzontale.
- Logging anonimo senza dati sensibili, e monitoraggio con alert su errori di sicurezza critici.
- Protezione server-side base contro CSRF/XSS tramite sanificazione input; nessun frontend.
- Nessuna gestione ruoli o permessi, solo controllo autenticazione utente.
- Politiche di rotazione e gestione chiave JWT da definire e formalizzare.

## 3. Architettura backend
Architettura a microservizio o monolite modulare stateless, basata su REST API expose e database PostgreSQL. 
- Layer controller/handler per esposizione endpoint REST.
- Layer service per logica di business: registrazione, login, reset password.
- Repository per accesso DB con ORM (es. JPA/Hibernate).
- Middleware/Filtro per validazione token JWT su richieste protette.
- Componente di gestione sicurezza (hashing password, gestione JWT).
- Componente integrazione email asincrona per reset password.
- Modulo logging e monitoraggio centralizzato.
- Possibilità di scalare orizzontalmente replicando istanze senza stato condiviso.
- Bear integrato come libreria/framework di supporto per sicurezza e logging (da approfondire).

## 4. Moduli e responsabilità
- Modulo User Management: registrazione e gestione dati utente.
- Modulo Auth: login, generazione e validazione token JWT.
- Modulo Password Reset: gestione codice temporaneo, invio email, reset password.
- Modulo Security: hashing password, gestione token, mitigazione brute force.
- Modulo Middleware: validazione JWT per protezione endpoint.
- Modulo Logging & Monitoring: logging eventi anonimi, metriche, alert.
- Modulo Configurazione: gestione parametri secret key, rate limiting, email.
- Modulo Bear Integration: wrapper o adapter per utilizzare Bear nel backend.

## 5. API principali
### POST /api/v1/auth/register
- Request body schema: 
  - email (string, required, valid email format, unique)
  - password (string, required, min 8 chars, complexity enforced)
- Response body schema:
  - 201 Created: message: string ("User registered successfully")
  - 400 Bad Request: errors: array (validation errors)
  - 409 Conflict: message: string ("Email already exists")
- Esempio request:
```json
{
  "email": "user@example.com",
  "password": "Password123!"
}
```
- Esempio response 201:
```json
{
  "message": "User registered successfully"
}
```

### POST /api/v1/auth/login
- Request body schema:
  - email (string, required)
  - password (string, required)
- Response body schema:
  - 200 OK: 
    - token (string, JWT token)
    - expires_in (integer, seconds until token expiration)
  - 400 Bad Request: validation errors
  - 401 Unauthorized: message: string ("Invalid credentials")
- Esempio request:
```json
{
  "email": "user@example.com",
  "password": "Password123!"
}
```
- Esempio response 200:
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expires_in": 3600
}
```

### GET /api/v1/protected/resource
- Protected endpoint example
- Headers:
  - Authorization: Bearer <JWT_TOKEN>
- Response:
  - 200 OK: resource data JSON
  - 401 Unauthorized: message "Token invalid or expired"

### POST /api/v1/auth/reset-password/request
- Request body schema:
  - email (string, required)
- Response body schema:
  - 200 OK: message "Password reset email sent if account exists"
  - 400 Bad Request: validation errors
- Esempio request:
```json
{
  "email": "user@example.com"
}
```
- Esempio response 200:
```json
{
  "message": "Password reset email sent if account exists"
}
```

### POST /api/v1/auth/reset-password/confirm
- Request body schema:
  - reset_code (string, required, alphanumeric, unique, temporaneo)
  - new_password (string, required, min 8 chars, complexity)
- Response body schema:
  - 200 OK: message "Password reset successful"
  - 400 Bad Request: validation errors, code expired/invalid
- Esempio request:
```json
{
  "reset_code": "ABC123XYZ",
  "new_password": "NewPassword123!"
}
```
- Esempio response 200:
```json
{
  "message": "Password reset successful"
}
```

## 6. Business logic
- Registrazione: validazione formale email/password, verifica unicità email, hashing password con Argon2/bcrypt, salvataggio utente.
- Login: verifica email esiste, controllo password hash, generazione token JWT con claims email/id, impostazione scadenza 1h, restituzione token.
- Autenticazione middleware: estrae token JWT, verifica firma e scadenza, autorizza accesso endpoint.
- Reset password: generazione codice reset unico e temporaneo, memorizzazione associata all’utente, invio email con link + codice, validazione codice reset alla conferma, aggiornamento password hashata, invalidamento codice reset.
- Mitigazione brute force: contatori tentativi login/reset con blocco temporaneo dopo soglia (parametrico).
- Logging: registrazione anonima eventi di accesso, login falliti, reset avviati e conclusi, senza dati sensibili.
- Monitoraggio: raccolta metriche base (errori, login rate), alert su anomalie e errori critici da configurare.

## 7. Persistenza e integrazioni
- PostgreSQL per:
  - Tabella utenti: id, email, password_hash, created_at, updated_at
  - Tabella reset_password: user_id, reset_code, expiry_timestamp, used_flag
- Integrazione Bear: libreria/framework di supporto per routine sicurezza e logging (da definire).
- Servizio di posta elettronica esterno configurabile per invio email reset password.
- Storage e gestione sicura delle secret key JWT tramite sistema di configurazione sicuro (es. vault, environment variables).

## 8. Autenticazione e autorizzazione
- Autenticazione stateless con token JWT firmato HMAC SHA256.
- JWT include claims: userId, email, issuedAt, expiration.
- Secret key robusta, lunga e rotabile.
- Middleware di validazione JWT per accesso a endpoint protetti: verifica firma e expiration, rifiuta token scaduti o invalidi.
- Nessuna autorizzazione basata su ruoli o permessi.
- Limitazione tentativi login/reset per mitigazione attacchi brute force.

## 9. Gestione errori
- Risposte HTTP con codici appropriati:
  - 400 Bad Request per input non valido o errori di validazione.
  - 401 Unauthorized per token assenti, invalidi o scaduti.
  - 403 Forbidden non previsto (no ruoli).
  - 404 Not Found per risorse non esistenti (es. codice reset non valido).
  - 409 Conflict per email duplicata in registrazione.
  - 500 Internal Server Error per errori imprevisti.
- Error messages chiari ma non dettagliati per evitare esposizione dati.
- Logging degli errori lato server senza dati sensibili.
- Standard JSON error response con campo message e dettagli errori opzionali.

## 10. Strategia di test backend
- Test_register_user_success (unit): verifica registrazione con dati validi, output 201 e user creato.
- Test_register_user_duplicate_email (unit): verifica errore 409 su email esistente.
- Test_login_valid_credentials (unit): verifica login con credenziali corrette e token JWT generato.
- Test_login_invalid_password (unit): verifica 401 login fallito.
- Test_jwt_validation_middleware (integration): verifica accesso endpoint protetto con token valido/scaduto/assente.
- Test_reset_password_request_existing_email (unit): verifica generazione codice reset e invio email fittizio.
- Test_reset_password_confirm_valid_code (unit): verifica reset password riuscito con codice valido.
- Test_rate_limiting_bruteforce (integration): simula multipli tentativi falliti e blocco temporaneo.
- Test_hashing_password (unit): verifica hashing sicuro con Argon2/bcrypt e confronto sicuro.
- Test_logging_anonymous_events (integration): verifica che eventi di login/reset vengano loggati senza dati sensibili.
- Test_performance_under_load (e2e): simulazione molteplici richieste login e registrazione per testing scalabilità.
- Test_security_csrf_xss_injection (integration): verifica sanitizzazione e assenza di vulnerabilità di base.
- Test_email_service_timeout_handling (integration): verifica comportamento reset password con timeout email.
- Test_jwt_secret_rotation (unit): verifica invalidazione token vecchi e generazione nuovi token post rotazione.

## 11. Rischi tecnici
- Ritardi di sviluppo dovuti a incertezza sull’integrazione Bear.
- Parametri non definiti o troppo restrittivi per rate limiting e blocco login potrebbero impattare usabilità.
- Dipendenza da sistema email esterno per reset password può causare ritardi o perdite di email critiche.
- Eventuali lock DB e concorrenza in ambiente cluster PostgreSQL durante accessi simultanei devono essere validati.
- Mancanza di gestione ruoli potrebbe richiedere refactoring futuro in caso di evoluzione requisito.
- Esposizione accidentale di dati sensibili nei log o gestione inappropriata delle secret key JWT possono compromettere sicurezza.
- Assenza di definizione policy rotazione chiavi JWT crea rischio di vulnerabilità a lungo termine.
- Protezioni CSRF/XSS basate solo su sanificazione input devono essere valutate approfonditamente con test mirati.
- Scalabilità orizzontale deve essere testata realisticamente per identificare possibili colli di bottiglia.

## 12. Struttura file proposta
``` 
auth-backend/
  Application.java                     # Entry point — public static void main(String[] args)
  config/
    Settings.java                     # App config — class Settings (JWT secret, email config, rate limiting params)
  models/
    User.java                        # JPA Entity User — class User (id, email, passwordHash, timestamps)
    PasswordResetToken.java          # Entity reset token — class PasswordResetToken (userId, code, expiry, used)
    dto/
      UserRegisterRequest.java       # DTO registration request — class UserRegisterRequest(email, password)
      UserLoginRequest.java          # DTO login request — class UserLoginRequest(email, password)
      JwtResponse.java               # DTO login response — class JwtResponse(token, expiresIn)
      PasswordResetRequest.java     # DTO reset request — class PasswordResetRequest(email)
      PasswordResetConfirm.java     # DTO reset confirm — class PasswordResetConfirm(resetCode, newPassword)
  repositories/
    UserRepository.java              # JPA Repository User — interface UserRepository extends JpaRepository<User, Long>
    PasswordResetRepository.java    # Repository reset token
  services/
    UserService.java                 # Business logic user reg. and lookup — registerUser(), findByEmail()
    AuthService.java                 # Login, JWT gen/validate — login(), generateJwtToken(), validateJwt()
    PasswordResetService.java        # Reset token gen, validation, password update — createResetToken(), resetPassword()
    EmailService.java                # Service invio email reset — sendResetPasswordEmail()
    SecurityService.java             # Hashing password, brute force mitigation — hashPassword(), verifyPassword(), rateLimitCheck()
  controllers/
    AuthController.java              # REST endpoints auth — register(), login(), requestResetPassword(), confirmResetPassword()
    ProtectedController.java         # Example protected endpoint — getProtectedResource()
  middleware/
    JwtAuthenticationFilter.java    # Middleware per validazione JWT su richieste protette
    RateLimitingFilter.java          # Filter base per rate limiting e blocco temporaneo
  integration/
    BearAdapter.java                # Classe wrapper/adattatore per integrazione Bear (da dettagliare)
  logging/
    LoggerConfig.java               # Configurazione logging anonimo sicuro
    EventLogger.java                # Classe per logging eventi accesso/login/reset anonimi
  monitoring/
    MetricsCollector.java           # Raccolta metriche base
    AlertService.java               # Alert su anomalie/errori critici
  exceptions/
    CustomException.java            # Gestione errors customizzati con codici HTTP e messaggi standard
  tests/
    auth/
      UserServiceTest.java          # Unit test UserService — test_register_user_success, test_register_user_duplicate_email
      AuthServiceTest.java          # Unit test AuthService — test_login_valid_credentials, test_login_invalid_password
      PasswordResetServiceTest.java # Unit test PasswordResetService
      RateLimitingTest.java         # Integration test per rate limiting e brute force
      JwtAuthenticationFilterTest.java # Integration test middleware
    integration/
      EmailServiceIntegrationTest.java # Simulazione sistema email esterno
      PerformanceLoadTest.java      # E2E performance e scalabilità
      SecurityVulnerabilityTests.java # Test CSRF, XSS, injection
```

## 13. Piano di implementazione
1. Definire dettagli tecnici e modalità integrazione Bear con team responsabile.
2. Progettare e realizzare schema DB PostgreSQL per User e PasswordResetToken.
3. Implementare moduli modello dati e repository JPA.
4. Realizzare service UserService (registrazione, controllo unicità) con hashing password.
5. Implementare AuthService con login, verifica, generazione JWT, e middleware JwtAuthenticationFilter.
6. Creare controller REST AuthController con endpoint register/login.
7. Implementare PasswordResetService, EmailService per gestione reset password e invio email.
8. Realizzare endpoint REST reset-password (request e confirm).
9. Integrare mitigazione brute force e filtro RateLimitingFilter, parametrizzato.
10. Configurare logging anonimo e sicuro tramite EventLogger e LoggerConfig.
11. Implementare monitoring base con raccolta metriche e AlertService.
12. Scrivere test unitari e di integrazione per moduli critici e simulazioni realistiche.
13. Eseguire test di carico e stress per verificare scalabilità orizzontale e stabilità.
14. Documentare backend e API con esempi di richiesta e risposta.
15. Validare sicurezza con test CSRF/XSS e eventuali penetration test.
16. Pianificare gestione rotazione secret key JWT e procedure di failover per email.
17. Rilasciare MVP stabilendo parametri rate limiting definitivi e chiarendo punti aperti Bear.