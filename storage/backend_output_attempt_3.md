MODULE: BACKEND VERSION: 1
## 1. Obiettivo backend
Sviluppare un backend Java-based utilizzando il framework Bear per fornire servizi REST API di autenticazione utenti. Il sistema deve consentire registrazione, login tramite email e password, gestione sicura di token JWT per accesso protetto agli endpoint, e un flusso funzionale di reset password tramite email con token univoco. Garantire sicurezza, affidabilità e logging per operazioni critiche.

## 2. Assunzioni tecniche
- Stack: Java, Bear framework per REST API, PostgreSQL come database, JWT per autenticazione.
- Password salvate in forma hashed (bcrypt).
- Token JWT con durata configurabile (15-60 minuti) e firma sicura.
- Infrastruttura email esterna disponibile per invio token reset password.
- HTTPS è presupposto per la trasmissione sicura dei dati.
- Endpoint protetti da middleware Bear che verifica validità token.
- Limitazione tentativi login per mitigare attacchi brute force.
- Mancata gestione ruoli utente complessi e meccanismi refresh token avanzati.
- Gestione errori database prevista con risposta controllata e fallback retry per email.

## 3. Architettura backend
Architettura a 3 livelli:
- Presentation layer: Bear REST API con endpoint specifici per registrazione, login, reset password; protetti da middleware autenticazione JWT.
- Business logic layer: servizi per gestione utenti, autenticazione, token JWT, politiche di sicurezza e logging.
- Data access layer: repository PostgreSQL con accesso tramite DAO o ORM leggero; gestisce utenti, password hash, token reset.
Componenti principali:
- JwtProvider: creazione e validazione token JWT con gestione firma e durata.
- AuthenticationService: logica login, registrazione, verifica credenziali, limitazione tentativi login.
- ResetPasswordService: gestione token reset, invio email, validazione token, cambio password.
- UserRepository: gestione persistenza utenti e campi associati.
- LoggingService: logging strutturato su accessi, errori e operazioni sensibili.
- Bear Middleware: interceptor per validazione token JWT su chiamate protette.

## 4. Moduli e responsabilità
- Modulo User: gestione modello utente, validazione dati, hashing password.
- Modulo Auth:
  - Endpoint Register: registrazione utente, validazione input, check unicità email, hashing password.
  - Endpoint Login: verifica credenziali, blocco tentativi errati, generazione token JWT.
  - Middleware JWT: intercettazione chiamate per verifica token JWT.
- Modulo ResetPassword:
  - Endpoint Richiesta reset: genera token univoco, invia email con token.
  - Endpoint Cambio password: riceve token, valida, aggiorna password hashed.
- Modulo Persistence: definizione schema DB, gestione query utenti e token.
- Modulo Logging: registrazione eventi di sicurezza, accessi, errori.
- Modulo Config: impostazioni configurabili (durata token, parametri sicurezza).
  
## 5. API principali
- POST /api/v1/register
  - Request body:
    ```json
    {
      "email": "string (valid email format, max 255 chars)",
      "password": "string (min 8 chars, complessità)"
    }
    ```
  - Response body:
    - 201 Created
    ```json
    {
      "message": "User registered successfully"
    }
    ```
    - 400 Bad Request (validation fail)
    - 409 Conflict (email already exists)
  
- POST /api/v1/login
  - Request body:
    ```json
    {
      "email": "string (valid email)",
      "password": "string"
    }
    ```
  - Response body:
    - 200 OK
    ```json
    {
      "token": "jwt_token_string",
      "expires_in": "integer (seconds)"
    }
    ```
    - 401 Unauthorized (invalid credentials)
    - 429 Too Many Requests (blocked for brute force limit)
  
- POST /api/v1/reset-password/request
  - Request body:
    ```json
    {
      "email": "string (valid email)"
    }
    ```
  - Response body:
    - 200 OK
    ```json
    {
      "message": "Reset token sent if email exists"
    }
    ```
  
- POST /api/v1/reset-password/confirm
  - Request body:
    ```json
    {
      "reset_token": "string",
      "new_password": "string (min 8 chars, complessità)"
    }
    ```
  - Response body:
    - 200 OK
    ```json
    {
      "message": "Password updated successfully"
    }
    ```
    - 400 Bad Request (invalid or expired token)
  
- Tutti gli endpoint diversi da /register e /login sono protetti tramite middleware JWT che verifica token nel header Authorization Bearer.

## 6. Business logic
- Registrazione: validazione email e password, verifica unicità, hashing bcrypt password, salvataggio utente.
- Login: validazione input, ricerca utente, verifica hash password, limitazione tentativi falliti con blocco temporaneo, generazione token JWT temporizzato.
- JWT: generazione con firma (algoritmo sicuro), durata configurabile, validazione middleware su richieste protette.
- Reset password: creazione token univoco (random sicuro), salvataggio associato all’utente con scadenza, invio email con token, verifica token e cambio password hashed.
- Logging: registrazione eventi con informazioni su accessi, errori autenticazione, reset password.
- Gestione errori database: fallback retry per operazioni critiche (es. invio email), risposte 500 standardizzate in caso di disservizio.
- Protezione brute force oltre blocco tentativi base da dettagliare e documentare (azione QA).

## 7. Persistenza e integrazioni
- Database: PostgreSQL
- Tabella utenti:
  - id (PK, UUID)
  - email (varchar unique, indexed)
  - password_hash (varchar)
  - reset_token (varchar, nullable)
  - reset_token_expiry (timestamp, nullable)
  - failed_login_attempts (integer, default 0)
  - last_failed_login (timestamp, nullable)
- Repository con query parametrizzate per evitare injection.
- Integrazione con sistema esterno SMTP/email service per invio token reset con retry e fallback.
- Configurazione parametri DB, email e sicurezza centralizzata.
- Migrazione schema: da implementare con tool standard (Flyway/Liquibase), attualmente solo menzionata, da dettagliare.

## 8. Autenticazione e autorizzazione
- Autenticazione tramite JWT standard:
  - Token generato alla login firmato con chiave segreta.
  - Header Authorization: Bearer token.
  - Middleware Bear con intercettazione chiamate per verifica token e controllo scadenza.
- Autorizzazione minima: ruolo utente non previsto, tutti gli utenti autenticati accedono a endpoint protetti.
- Limitazione tentativi login per mitigare brute force (block temporaneo dopo N tentativi).
- Reset password senza autenticazione ma con token univoco inviato via email.
- Gestione sicura delle chiavi JWT (storage e accesso protetti).

## 9. Gestione errori
- Validazione input con risposte 400 e dettagli errori (JSON uniforme).
- 401 per credenziali invalidi o token mancanti/non validi.
- 429 per blocco tentativi eccessivi login.
- 404 per risorse non trovate (se applicabile).
- 500 per errori interni gestiti con logging e messaggi generici.
- Timeout e retry per operazioni email.
- Gestione race condition su reset token tramite locking/transazioni.
- Log dettagliati di errori e accessi anomali.

## 10. Strategia di test backend
- test_register_user_success
  - Tipo: unit/integration
  - Verifica registrazione con email valida, password corretta, email unica.
  - Input: email/password validi; Expected output: 201 Created + utente creato.
- test_register_user_duplicate_email
  - Tipo: unit
  - Verifica blocco registrazione email duplicata.
  - Input: email già presente; Expected output: 409 Conflict.
- test_login_success
  - Tipo: unit/integration
  - Verifica login corretto con generazione token JWT.
  - Input: credenziali corrette; Expected output: 200 OK + token JWT.
- test_login_fail_block_bruteforce
  - Tipo: unit
  - Verifica blocco login dopo tentativi errati.
  - Input: credenziali errate N volte; Expected output: 429 Too Many Requests.
- test_jwt_validation_middleware
  - Tipo: integration
  - Verifica accesso protetto senza token/with invalid token.
  - Input: chiamata endpoint protetto con token mancante o scaduto; Expected output: 401 Unauthorized.
- test_reset_password_flow
  - Tipo: integration
  - Verifica invio token reset, ricezione email, cambio password valido.
  - Input: richiesta reset + uso token valido; Expected output: 200 OK password aggiornata.
- test_reset_password_invalid_token
  - Tipo: unit
  - Verifica rifiuto token reset invalido o scaduto.
  - Input: token errato; Expected output: 400 Bad Request.
- test_database_failure_handling
  - Tipo: integration
  - Simula failure DB e verifica risposte 500 prevedibili e fallback retry email.
- test_input_sanitization
  - Tipo: unit
  - Verifica validazione e sanitizzazione input per prevenire injection.

## 11. Rischi tecnici
- Dipendenza da servizio email esterno per reset password: rischio integrazione e disservizio.
- Gestione revoca token JWT limitata a scadenza, assenza blacklist o revoca manuale.
- Mancanza verifica email esplicita, potenziale account fraudolenti o non validi.
- Strategie anti brute force non ancora dettagliate oltre semplice blocco tentativi.
- Ambiguità futura nella gestione ruoli e permessi.
- Migrazione schema DB non dettagliata, rischio complessità evolutiva.
- Possibili race condition in concorrenza su reset token.
- Necessità di gestione errori DB robusta per garantire disponibilità e previsione fallimenti.

## 12. Struttura file proposta
``` 
project_root/
  Main.java                 # Entry point applicazione — public static void main(String[] args)
  config/
    Settings.java           # Configurazione app — class Settings { jwtDuration, dbConfig, emailConfig }
  models/
    User.java               # Modello User — class User { UUID id; String email; String passwordHash; ... }
    ResetToken.java         # Modello token reset — class ResetToken { String token; Instant expiry; }
  repositories/
    UserRepository.java     # Accesso DB utenti — interface UserRepository { findByEmail(), save(), updatePassword() }
  services/
    AuthenticationService.java  # Logica login, registrazione — boolean authenticate(String, String), String generateJwt(User)
    ResetPasswordService.java    # Gestione reset password — void createResetToken(), boolean validateToken(), void resetPassword()
    JwtProvider.java             # Creazione e validazione token JWT — String generateToken(), boolean validateToken()
    LoggingService.java          # Logging eventi — void logAccess(), void logError()
  middleware/
    JwtAuthMiddleware.java       # Middleware Bear per verifica JWT — boolean handle(Request)
  routers/
    AuthRouter.java             # Endpoint auth — void register(Request), void login(Request)
    ResetPasswordRouter.java    # Endpoint reset password — void requestReset(Request), void confirmReset(Request)
  tests/
    AuthControllerTest.java     # Test registrazione e login
    ResetPasswordServiceTest.java # Test flusso reset password
    JwtProviderTest.java       # Test generazione e validazione JWT
    DatabaseFailureTest.java    # Test gestione errori DB e fallback email
  utils/
    PasswordHasher.java         # Funzioni hashing bcrypt — String hash(String), boolean verify(String, String)
    EmailSender.java            # Invio email con retry — void sendEmail(String, String)
  migrations/
    V1__create_user_table.sql    # Script creazione tabella utenti e indici
``` 

## 13. Piano di implementazione
1. Progettazione schema DB dettagliato e script migrazione versionato (alta priorità).
2. Implementazione modelli dati e repository, inclusi indici e vincoli univoci.
3. Sviluppo servizio hashing password bcrypt.
4. Implementazione endpoint registrazione con validazione e hashing.
5. Implementazione login con limitazione tentativi, gestione errori e generazione JWT.
6. Realizzazione JwtProvider e middleware di protezione Bear.
7. Sviluppo flusso reset password con generazione token, invio email e conferma cambio password, con retry e fallback.
8. Setup LoggingService per eventi sicurezza e audit.
9. Test di unità e integrazione per tutte le funzionalità principali, inclusi scenari di errore, gestione concorrenza, e simulazione failure DB.
10. Affinamento strategie anti brute force secondo indicazioni QA.
11. Documentazione dettagliata delle API e delle strategie di sicurezza.
12. Programmazione future estensioni per verifica email, revoca token e gestione ruoli.