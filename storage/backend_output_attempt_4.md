MODULE: BACKEND VERSION: 1
## 1. Obiettivo backend
Realizzare un backend di autenticazione utenti sicuro, performante e scalabile basato su Java e Bear framework che esponga REST API per registrazione, login, protezione endpoint con token JWT, reset password base. Il sistema deve garantire persistenza su PostgreSQL, hashing sicuro delle password con salt, validazione input rigorosa, logging e monitoraggio degli eventi critici, e gestione errori coerente. È richiesto il supporto per scalabilità orizzontale e prestazioni con latenza inferiore a 1 secondo sugli endpoint critici.

## 2. Assunzioni tecniche
- Validazione email solo per unicità, senza conferma via link.
- Password con minimo 8 caratteri contenenti lettere, numeri e simboli.
- Token JWT firmati con durata circa 1 ora (possibile refresh futuro).
- Reset password base via codice/link temporaneo inviato per email.
- Limiti di tentativi login/reset per mitigare attacchi brute force implementati.
- Gestione concorrente della registrazione con unicità email garantita lato DB e logica.
- Logging sicuro cifrato e monitoraggio con alert per eventi critici.
- Nessuna UI frontend, solo backend API.
- Nessuna MFA, social login o autorizzazioni granulari.
- Architettura e codice modulari separando logica business, persistence e API.

## 3. Architettura backend
Il backend è strutturato in livelli:
- Controller (Bear framework Router): espone gli endpoint REST per interazione esterna e validazione primaria.
- Service layer: contiene la logica business, gestione crittografia password, generazione/verifica token JWT, gestione reset password, gestione concorrenza.
- Repository layer: integra con PostgreSQL usando DAO/Repository pattern per utenti, token e codici reset, gestione transazioni DB e vincoli.
- Middleware Bear: filtro per protezione endpoint con verifica token JWT e gestione errori 401.
- Logging e monitoring componenti integrati per raccolta eventi critici e health checks.
L’architettura supporta scalabilità stateless del backend e uso connessioni pool DB.

## 4. Moduli e responsabilità
- auth.controller: gestisce endpoint registrazione, login, logout, reset password.
- auth.service: logica di validazione password, hashing sicuro, generazione token JWT, gestione reset.
- auth.repository: interfaccia diretta con tabelle utenti, token, reset codes in PostgreSQL.
- auth.middleware: Bear middleware per autenticazione con JWT token.
- logging: configurazione e gestione sicura dei log eventi critici.
- monitoring: componenti e configurazioni per monitoraggio salute sistema e alert.
- config: gestione centralizzata configurazioni (DB, JWT secret, limiti tentativi).
- utils: funzioni di supporto come validazioni comuni, crittografia.
- tests: test unitari, integrazione, sicurezza.

## 5. API principali

1. Registrazione utente
- Metodo HTTP e path: POST /api/v1/auth/register
- Request body schema:
  ```json
  {
    "email": "string, formato email valida, unico",
    "password": "string, min 8, lettere, numeri, simboli"
  }
  ```
- Response body schema:
  - 201 Created
    ```json
    { "message": "User registered successfully" }
    ```
  - 400 Bad Request (validazione)
  - 409 Conflict (email già esistente)
- Esempio:
  Request:
  ```json
  {
    "email": "user@example.com",
    "password": "Passw0rd!"
  }
  ```
  Response 201:
  ```json
  {
    "message": "User registered successfully"
  }
  ```

2. Login
- Metodo HTTP e path: POST /api/v1/auth/login
- Request body schema:
  ```json
  {
    "email": "string, formato email valida",
    "password": "string"
  }
  ```
- Response body schema:
  - 200 OK
    ```json
    {
      "token": "string JWT",
      "expires_in": 3600
    }
    ```
  - 401 Unauthorized (credenziali errate)
- Esempio:
  Request:
  ```json
  {
    "email": "user@example.com",
    "password": "Passw0rd!"
  }
  ```
  Response 200:
  ```json
  {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expires_in": 3600
  }
  ```

3. Reset password richiesta
- Metodo HTTP e path: POST /api/v1/auth/reset-password/request
- Request body schema:
  ```json
  {
    "email": "string, formato email valida"
  }
  ```
- Response body schema:
  - 200 OK
    ```json
    { "message": "Reset password email sent if user exists" }
    ```
- Esempio:
  Request:
  ```json
  {
    "email": "user@example.com"
  }
  ```
  Response 200:
  ```json
  {
    "message": "Reset password email sent if user exists"
  }
  ```

4. Reset password conferma
- Metodo HTTP e path: POST /api/v1/auth/reset-password/confirm
- Request body schema:
  ```json
  {
    "email": "string, formato email valida",
    "code": "string, codice temporaneo ricevuto via email",
    "new_password": "string, min 8 con lettere, numeri e simboli"
  }
  ```
- Response body schema:
  - 200 OK
    ```json
    { "message": "Password changed successfully" }
    ```
  - 400 Bad Request (codice errato o scaduto)
- Esempio:
  Request:
  ```json
  {
    "email": "user@example.com",
    "code": "ABC123",
    "new_password": "NewPassw0rd!"
  }
  ```
  Response 200:
  ```json
  {
    "message": "Password changed successfully"
  }
  ```

5. Endpoint protetto esempio
- Metodo HTTP e path: GET /api/v1/protected/resource
- Headers: Authorization: Bearer <token JWT valido>
- Response body schema:
  - 200 OK con dati protetti
  - 401 Unauthorized se token mancante, invalido o scaduto

## 6. Business logic
- Registrazione: verifica unicità email in transazione DB, validazione password, hashing sicurezza con salt forte (ad es. bcrypt).
- Login: verifica email esistenza, controllo password con hash, generazione token JWT firmato con scadenza (circa 1 ora).
- Middleware: verifica presenza token JWT, validità firma, scadenza, e corrispondenza utente.
- Reset password: generazione codice temporaneo unico e ttl limitato, invio via email esterno, conferma aggiornamento password con nuova validazione e hashing.
- Limitazione tentativi login/reset tramite contatori temporali per mitigare brute force.
- Logging eventi critici: ogni login, reset e errore sensibile è loggato con livello adeguato.
- Gestione concorrenza per registrazione e reset password per evitare race condition.

## 7. Persistenza e integrazioni
- PostgreSQL con tabelle principali:
  - users: id (UUID PK), email (unique), password_hash, salt, created_at, updated_at.
  - reset_tokens: id PK, user_id FK, code (unique), expiration_time, used_flag.
- Indici per email, codice reset.
- Transazioni per operazioni critiche.
- Integrazione con SMTP esterno per invio email reset (retry e fallback da definire).
- Pool di connessioni DB per performance e scalabilità.
- Migrazioni schema previste tramite tool (liquibase/flyway), documentate e versionate.

## 8. Autenticazione e autorizzazione
- Autenticazione tramite token JWT:
  - Token firmati con secret sicuro lato server.
  - Payload ridotto con user_id, issue_at, expiry.
- Middleware Bear verifica token in header Authorization.
- Nessuna autorizzazione granulari o ruoli previsti.
- Accesso negato (401) se token non presente, scaduto o non valido.

## 9. Gestione errori
- Risposte con codici HTTP appropriati (400, 401, 409).
- Messaggi di errore chiari ma senza esposizione dati sensibili.
- ValidationException per input errati.
- Gestione timeout e errori DB con retry limitati.
- Error handling centralizzato e logging sicuro degli errori.
- Payload error consistente con fields "error_code", "message".

## 10. Strategia di test backend
- test_register_user_success (integration): verifica registrazione con email unica e password valida, attesa 201.
- test_register_user_duplicate_email (integration): tentativo duplicato genera 409.
- test_login_success (integration): login valido restituisce JWT e expire.
- test_login_invalid_password (integration): password errata ritorna 401.
- test_reset_password_request_existing_email (integration): generazione codice e invio email simulato.
- test_reset_password_request_non_existing_email (integration): risposta 200 senza esporre info.
- test_reset_password_confirm_success (integration): cambio password con codice valido.
- test_reset_password_confirm_invalid_code (integration): errore 400.
- test_jwt_protected_endpoint_access (integration): accesso con token valido 200, senza o scaduto 401.
- test_password_hashing_correctness (unit): verifica hash con salt produce risultati attesi.
- test_input_validation (unit): validazione formati email e complessità password.
- test_brute_force_limit_login (integration): simulazione tentativi multipli blocco.
- test_logging_events (integration): verifica log eventi critici scritti correttamente.
- test_concurrent_registrations (integration): verifica unicità e assenza race condition.
- test_db_migrations (integration): applicazione migrazioni corrette.
- test_error_handling_consistency (unit): verifica uniformità formato errori.

## 11. Rischi tecnici
- Mancata conferma email può causare account non validi.
- Gestione concorrente registrazioni con stessa email richiede forte controllo DB e lock.
- Assenza meccanismo refresh token limita UX.
- Vulnerabilità brute force se limiti tentativi o monitoraggio insufficienti.
- Dipendenza da sistema email esterno senza strategia fallback può bloccare reset password.
- Gestione logging e monitoraggio deve rispettare compliance sicurezza.
- Scalabilità orizzontale richiede configurazione stateless e pooling DB.
- Assenza autorizzazioni granulari limita futuri evoluzioni.

## 12. Struttura file proposta
```
project_name/
  Main.java                # Entry point app Bear framework - public class Main { public static void main(String[] args) }
  config/
    Settings.java          # Configurazione app - class Settings { DB config, JWT secret, limiti tentativi }
  models/
    User.java              # Modello utente - class User {UUID id, String email, String passwordHash, String salt, timestamps}
    ResetToken.java        # Modello reset password - class ResetToken {UUID id, UUID userId, String code, Date expiration, Boolean used}
  repository/
    UserRepository.java    # Interazione DB utenti - interface UserRepository {User findByEmail(), void save(User)}
    ResetTokenRepository.java # Interazione DB reset token
  service/
    AuthService.java       # Logica auth - register(), login(), generateJWT(), verifyJWT(), resetPasswordRequest(), resetPasswordConfirm()
    ValidationService.java # Validazioni input - validateEmail(), validatePassword()
    PasswordHasher.java    # Hash password con salt sicuro - hash(), verify()
    EmailService.java      # Invio email reset con retry - sendResetEmail()
    BruteForceProtectionService.java # Conta tentativi e blocchi login/reset
  controller/
    AuthController.java    # Endpoint REST - register(), login(), resetPasswordRequest(), resetPasswordConfirm()
  middleware/
    JwtAuthMiddleware.java # Bear middleware verifica JWT token
  logging/
    LoggerConfig.java      # Configurazione logging sicuro eventi critici
    EventLogger.java       # Logger eventi login/reset/errore
  monitoring/
    MonitorService.java    # Health check e alert base
  utils/
    Utils.java             # Funzioni ausiliarie (es. generatore codice alfanumerico)
  tests/
    AuthServiceTest.java       # Unit + integration test AuthService
    AuthControllerTest.java    # Test endpoint REST
    PasswordHasherTest.java    # Unit test hashing password
    BruteForceProtectionTest.java # Test limit brute force
    ConcurrencyTest.java       # Test registrazioni concorrenti
    LoggingTest.java           # Test logging eventi
```

## 13. Piano di implementazione
1. Definizione e creazione schema DB utenti e reset token, configurazione migrazioni.
2. Implementazione modelli dati User e ResetToken.
3. Sviluppo UserRepository e ResetTokenRepository con supporto transazioni.
4. Realizzazione PasswordHasher con algoritmo bcrypt e salt sicuro.
5. Implementazione AuthService: registrazione con controllo unicità email, hashing password, login con verifica e generazione JWT, reset password richiesta e conferma, limitazione tentativi brute force.
6. Sviluppo AuthController con endpoint REST, validazione input tramite ValidationService.
7. Configurazione JwtAuthMiddleware per protezione endpoint con gestione errori 401.
8. Implementazione EmailService per invio email reset password con retry e fallback di base.
9. Setup LoggerConfig e EventLogger per logging eventi critici e errori.
10. Implementazione MonitorService per health check e alert semplici.
11. Realizzazione test unitari e di integrazione su moduli principali.
12. Esecuzione test di sicurezza, prestazioni e concorrenza.
13. Revisione e ottimizzazione per performance e scalabilità.
14. Preparazione documentazione tecnica API, configurazioni sicurezza, istruzioni deploy e utilizzo.
15. Deliverable finale con codice, test report, documentazione e configurazione monitoraggio.