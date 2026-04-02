MODULE: BACKEND VERSION: 1

## 1. Obiettivo backend
Realizzare un backend modulare in Java utilizzando Bear framework che gestisca l’autenticazione utenti tramite REST API. Le funzionalità principali devono includere la registrazione utenti con validazione email unica e password conforme a policy, login con generazione di token JWT valido per 1 ora, protezione di endpoint tramite filtro JWT, gestione della richiesta e conferma reset password tramite token temporanei validi 15 minuti con invio email tramite sistema esterno, e logging di eventi critici. Il sistema deve garantire sicurezza elevata, performance entro 200ms per login e registrazione e scalabilità orizzontale.

## 2. Assunzioni tecniche
- Policy password: minimo 8 caratteri, contenente lettere e numeri.
- Token JWT con scadenza impostata a 1 ora.
- Token reset password con validità 15 minuti.
- Persistenza dati utenti, token reset e log eventi in PostgreSQL.
- Password memorizzate con hashing e salting tramite bcrypt.
- Comunicazioni via HTTPS.
- Invio email tramite sistema esterno integrato, non gestito direttamente nel codice.
- Gestione blocco tentativi login multipli falliti e mitigazioni CSRF implementate a livello applicativo.
- Nessuna implementazione iniziale di multi-fattore, ruoli, logout esplicito o revoca token.
- Logging limitato ad eventi critici di autenticazione e reset password.

## 3. Architettura backend
Architettura modulare layerizzata:
- Controller/Router layer: espone REST API per autenticazione.
- Service layer: contiene logica di business per registrazione, login, reset password.
- Persistence layer: accesso a PostgreSQL tramite DAO/repository.
- Security layer: gestione hashing password, generazione e validazione JWT, filtro Bear per protezione endpoint.
- Integration layer: chiamate verso sistema esterno per invio email reset password.
Sistema stateless, progettato per scalabilità orizzontale e prestazioni <200ms su login/registrazione.
Error handling centralizzato con codici HTTP appropriati.
Logging eventi critici per audit e monitoraggio base.

## 4. Moduli e responsabilità
- UserModule: gestione dati utenti, validazioni registrazione e login.
- AuthModule: gestione login, generazione e validazione token JWT.
- ResetPasswordModule: generazione token reset, invio email, cambio password con token.
- SecurityModule: hashing/salting password con bcrypt, filtro Bear per protezione endpoint JWT.
- PersistenceModule: repository per utenti, token reset, log eventi.
- EmailIntegrationModule: interfaccia per invio email via sistema esterno.
- LoggingModule: logging audit eventi critici autenticazione e reset.
- ErrorHandlingModule: gestione centralizzata errori e risposte API.

## 5. API principali

### POST /api/v1/auth/register
- Request body:
  ```json
  {
    "email": "string (email valida, univoca)",
    "password": "string (min 8 caratteri, lettere e numeri)"
  }
  ```
- Response body:
  - 201 Created
    ```json
    {
      "message": "Utente registrato con successo"
    }
    ```
  - 400 Bad Request (validation error)
  - 409 Conflict (email già registrata)
- Esempio request:
  ```json
  {
    "email": "utente@example.com",
    "password": "Pass1234"
  }
  ```
- Esempio response 201:
  ```json
  {
    "message": "Utente registrato con successo"
  }
  ```

### POST /api/v1/auth/login
- Request body:
  ```json
  {
    "email": "string (email valida)",
    "password": "string"
  }
  ```
- Response body:
  - 200 OK
    ```json
    {
      "token": "string (JWT, valido 1 ora)"
    }
    ```
  - 400 Bad Request (validation error)
  - 401 Unauthorized (credenziali errate, blocco tentativi)
- Esempio request:
  ```json
  {
    "email": "utente@example.com",
    "password": "Pass1234"
  }
  ```
- Esempio response 200:
  ```json
  {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
  ```

### POST /api/v1/auth/reset-password/request
- Request body:
  ```json
  {
    "email": "string (email valida)"
  }
  ```
- Response body:
  - 200 OK
    ```json
    {
      "message": "Email per reset password inviata se email registrata"
    }
    ```
- Esempio request:
  ```json
  {
    "email": "utente@example.com"
  }
  ```
- Esempio response:
  ```json
  {
    "message": "Email per reset password inviata se email registrata"
  }
  ```

### POST /api/v1/auth/reset-password/confirm
- Request body:
  ```json
  {
    "token": "string (token reset valido)",
    "newPassword": "string (min 8 caratteri, lettere e numeri)"
  }
  ```
- Response body:
  - 200 OK
    ```json
    {
      "message": "Password modificata con successo"
    }
    ```
  - 400 Bad Request (validation error)
  - 401 Unauthorized (token scaduto o non valido)
- Esempio request:
  ```json
  {
    "token": "reset-token-string",
    "newPassword": "NewPass123"
  }
  ```
- Esempio response 200:
  ```json
  {
    "message": "Password modificata con successo"
  }
  ```

### Protezione endpoint API
- Middleware/filtro Bear che valida JWT.
- Endpoint protetti restituiscono 401 Unauthorized se token mancante o non valido.

## 6. Business logic
- Registrazione utente con controllo univocità email, validazione password e hashing bcrypt.
- Login con verifica password hashed, limitazione tentativi falliti e generazione JWT.
- Gestione token JWT con scadenza 1 ora per sessioni stateless.
- Reset password: generazione token unico temporaneo 15 minuti, salvataggio nel DB, invio email tramite sistema esterno.
- Conferma reset password con verifica token e aggiornamento password hashed.
- Blocchi e mitigazioni per tentativi login falliti multiple per prevenire brute force.
- Logging eventi critici quali registrazione, login, reset e errori rilevanti.

## 7. Persistenza e integrazioni
- PostgreSQL con tabelle:
  - users: id, email (unique), password_hashed, created_at, updated_at.
  - reset_tokens: id, user_id, token, expiration_timestamp.
  - auth_logs: id, user_id, event_type (login, reset, error), timestamp, ip_address.
- Integrazione email tramite modulo dedicato che espone interfaccia per invio email di reset.
- Connessioni DB configurate con pool e ottimizzate per performance.
- Bear framework integrato con DAO/repository per accesso DB.

## 8. Autenticazione e autorizzazione
- Autenticazione tramite email e password per login.
- Hashing/salting password con bcrypt per sicurezza.
- Generazione token JWT con algoritmo HMAC SHA256 e segreto configurato.
- Filtro Bear che valida token JWT per esplorare e permettere accesso solo a utenti autenticati.
- Nessuna gestione ruoli o permessi avanzati prevista.
- Token reset password generati univoci, temporanei e memorizzati per verifica.
- Protezione endpoint API tramite filtro JWT Bear.

## 9. Gestione errori
- Risposte API coerenti con codici HTTP:
  - 200 OK per successo
  - 201 Created per registrazione riuscita
  - 400 Bad Request per dati non validi
  - 401 Unauthorized per autenticazione fallita o token invalido
  - 409 Conflict per email già registrata
  - 500 Internal Server Error per errori imprevisti
- Messaggi di errore chiari e non eccessivamente descrittivi per sicurezza.
- Gestione edge cases: token scaduti, token manomessi, tentativi login multipli falliti con blocco temporaneo.
- Centralizzazione gestione errori in middleware o handler per consolidare logica.
- Logging errori e eventi critici per audit e monitoraggio.

## 10. Strategia di test backend

### UserModule - Registrazione
- test_register_user_success (integration)
  - Verifica registrazione con email e password validi
  - Input: email valida, password conforme
  - Output: HTTP 201 con messaggio successo
- test_register_user_duplicate_email (integration)
  - Verifica errore su email già registrata
  - Input: email duplicata
  - Output: HTTP 409 Conflict

### AuthModule - Login
- test_login_success (integration)
  - Verifica login con credenziali corrette
  - Input: email e password corrette
  - Output: HTTP 200, token JWT valido
- test_login_fail_wrong_password (integration)
  - Verifica errore credenziali errate
  - Input: password errata
  - Output: HTTP 401 Unauthorized
- test_login_block_after_multiple_failures (integration)
  - Verifica blocco login dopo tentativi falliti multipli
  - Input: 5 tentativi falliti consecutivi
  - Output: HTTP 401 con blocco attivo

### ResetPasswordModule
- test_reset_password_request_exists_email (integration)
  - Verifica invio email corretto se email esiste
  - Input: email registrata
  - Output: HTTP 200 messaggio invio
- test_reset_password_request_nonexistent_email (integration)
  - Verifica risposta identica se email non esiste (per sicurezza)
  - Input: email non presente
  - Output: HTTP 200 messaggio invio
- test_reset_password_confirm_success (integration)
  - Verifica cambio password con token reset valido
  - Input: token valido, nuova password conforme
  - Output: HTTP 200 messaggio successo
- test_reset_password_confirm_expired_token (integration)
  - Verifica errore con token scaduto
  - Input: token scaduto
  - Output: HTTP 401 Unauthorized

### Security and Performance
- test_password_hashing_security (unit)
  - Verifica password non memorizzata in chiaro
- test_jwt_token_generation_and_validation (unit)
  - Verifica corretto funzionamento creazione e validazione token
- test_api_response_time_login_register (performance)
  - Input: usual load request login e register
  - Output: risposta < 200ms

### Error handling
- test_error_responses_on_invalid_inputs (integration)
  - Input: dati malformati / token manomessi
  - Output: codici HTTP 400, 401 appropriati

### Integration Tests
- test_email_sending_mocked (integration)
  - Verifica invio email con sistema email esterno simulato

## 11. Rischi tecnici
- Dipendenza da sistema esterno email, con possibilità rilievi ritardi o vulnerabilità esterne.
- Mancanza di revoca esplicita token JWT o logout può consentire uso di token compromessi fino alla scadenza.
- Rischio denial of service legato a blocchi tentativi login se non configurato con adeguata tolleranza.
- Policy password base potrebbe non coprire scenari di sicurezza avanzati futuri.
- Possibilità di attacchi replay sui token JWT non mitigati a livello MVP.
- Limitato monitoraggio e alerting potrebbe risultare insufficiente in produzione.
- Mancanza gestione ruoli limita futura estendibilità funzionale.

## 12. Struttura file proposta
```plaintext
auth-backend/
  Main.java                   # Entry point dell'applicazione - public static void main(String[] args)
  config/
    Settings.java             # Configurazioni applicative - class Settings
  models/
    User.java                 # Entità User - class User {id, email, passwordHash, createdAt}
    ResetToken.java           # Entità ResetToken - class ResetToken {id, userId, token, expiry}
    AuthLog.java              # Entità AuthLog - class AuthLog {id, userId, eventType, timestamp, ip}
  repository/
    UserRepository.java       # Accesso dati User - interface UserRepository {findByEmail, save, ...}
    ResetTokenRepository.java # Accesso dati ResetToken - interface ResetTokenRepository {...}
    AuthLogRepository.java    # Accesso dati log autenticazione - interface AuthLogRepository {...}
  service/
    UserService.java          # Logica business utenti - class UserService {register, validatePassword, ...}
    AuthService.java          # Login e generazione JWT - class AuthService {login, generateJwt, validateJwt}
    ResetPasswordService.java # Gestione reset password - class ResetPasswordService {requestReset, confirmReset}
    EmailService.java         # Integrazione invio email - interface EmailService {sendResetEmail}
  security/
    PasswordHasher.java       # Hashing password bcrypt - class PasswordHasher {hash, verify}
    JwtTokenProvider.java     # JWT gestione token - class JwtTokenProvider {generateToken, validateToken}
    JwtAuthFilter.java        # Filtro Bear per protezione endpoint - class JwtAuthFilter {doFilter}
  controllers/
    AuthController.java       # Endpoint REST autenticazione - class AuthController {register, login, resetRequest, resetConfirm}
  exceptions/
    ApiException.java         # Eccezioni personalizzate API - class ApiException
    ApiExceptionHandler.java  # Gestione centrale errori - class ApiExceptionHandler {handleExceptions}
  logging/
    AuthLogger.java           # Logging eventi critici autenticazione - class AuthLogger {logEvent}
  tests/
    AuthControllerTest.java   # Test funzionali endpoint auth
    UserServiceTest.java      # Test unitari UserService
    AuthServiceTest.java      # Test unitari AuthService
    ResetPasswordServiceTest.java # Test ResetPasswordService
    PerformanceTest.java      # Test performance login/registrazione
    IntegrationEmailMockTest.java # Test integrazione email simulata
```

## 13. Piano di implementazione
1. Definizione schema dati PostgreSQL per users, reset_tokens, auth_logs.
2. Implementazione modelli Entity e repository per accesso DB.
3. Implementazione UserService con validazioni e hashing password.
4. Implementazione AuthService con login, verifica password e JWT generation.
5. Configurazione Bear framework con JwtAuthFilter per protezione endpoint.
6. Implementazione AuthController con endpoint register, login e protezione.
7. Implementazione ResetPasswordService con generazione token reset, memorizzazione e invio email tramite EmailService.
8. Implementazione endpoint reset-password request e confirm in AuthController.
9. Implementazione blocco tentativi login falliti e mitigazioni CSRF a livello applicativo.
10. Implementazione logging evento critici autenticazione e reset password.
11. Gestione centralizzata errori con ApiException e ApiExceptionHandler.
12. Setup HTTPS a livello ambientale (documentazione da aggiornare).
13. Scrittura test unitari, integration, performance secondo piano test definito.
14. Verifica criteri acceptance, tuning blocchi login, e refactoring finale.
15. Documentazione architettura, API, e configurazioni sicurezza in ambiente produzione.