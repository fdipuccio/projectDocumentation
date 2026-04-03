MODULE: BACKEND VERSION: 1
## 1. Obiettivo backend
Realizzare un backend RESTful per la gestione dell’autenticazione utenti che consenta registrazione, login, protezione di endpoint con token JWT, e reset password base. Il backend supporta la persistenza dati con PostgreSQL e garantisce sicurezza, performance, scalabilità e monitoraggio secondo la specifica PM. Ideale per integrazione in sistemi enterprise con Bear framework e stack Java.

## 2. Assunzioni tecniche
- Java sarà il linguaggio di sviluppo principale.
- Bear framework sarà usato per implementare l’API REST.
- PostgreSQL sarà il database di persistenza per utenti e token.
- JWT sarà utilizzato per generare e validare i token di accesso, con durata temporale di circa un’ora.
- Password hashing userà algoritmo sicuro (es. bcrypt o Argon2) con salting.
- Validazione input rigorosa per email e password con regex e controlli di formato.
- Logging sicuro sarà realizzato a livello server per eventi critici (login, reset, errori).
- Middleware Bear gestirà protezione endpoint verificando la validità del token JWT.
- Sarà implementato un semplice sistema di rate limiting per login e reset password per mitigare brute force.
- Nessun frontend o UI incluso.
- Scalabilità orizzontale garantita tramite statelessness del backend e database clusterabile.
- Nessun supporto MFA o ruoli granulari.
- Reset password base tramite invio di link o codice temporaneo via email.

## 3. Architettura backend
- Architettura a servizi REST con Bear framework come server HTTP.
- Persistenza dati su PostgreSQL con schema dedicato per utenti e token/reset.
- JWT con firma HMAC o RSA per autenticazione stateless.
- Middleware Bear per intercettare e validare i token nelle richieste protette.
- Moduli per registrazione, login, reset password, validazione input, logging e monitoraggio.
- Componente email service di invio link/codice reset password (astratto, configurabile).
- Sistema di logging centralizzato e monitoraggio degli eventi critici.
- Scalabilità orizzontale tramite esecuzione di più istanze stateless dietro bilanciatore di carico.
- Meccanismi base anti-brute force implementati tramite contatori temporizzati.
- Gestione degli errori uniforme con risposte standardizzate JSON e codici HTTP.

## 4. Moduli e responsabilità
- UserModel: definizione entità User, validazione dati, hashing password.
- UserRepository: interfaccia e implementazione per operazioni CRUD User su PostgreSQL.
- AuthService: logica registro/login, controllo credenziali, generazione JWT.
- JwtMiddleware: intercetta richieste protette e valida token JWT.
- PasswordResetService: gestione richiesta reset password, generazione e invio codice/link.
- EmailService: invio di email (interface astratta, da implementare/configurare esternamente).
- InputValidation: funzioni e regole di validazione per email, password e codici.
- LoggingService: gestione sicura del logging eventi critici.
- RateLimiter: controllo e limitazione tentativi login/reset per IP/utente.
- ErrorHandling: normalizzazione errori, mapping su codici HTTP e response JSON.
- ApiRouter: definizione rotte REST per registrazione, login, reset e test token protetti.

## 5. API principali

### POST /api/v1/auth/register
- Request body schema:
  - email: string, formato email, obbligatorio, unico nel DB.
  - password: string, minimo 8 caratteri con lettere, numeri, simboli, obbligatorio.
- Response body schema:
  - 201 Created: { "id": UUID, "email": string }
  - 400 Bad Request: { "error": string }
  - 409 Conflict: { "error": "Email already exists" }
- Esempio request:
  ```json
  {
    "email": "utente@example.com",
    "password": "Pa$$w0rd123"
  }
  ```
- Esempio response 201:
  ```json
  {
    "id": "8d9f1a70-1234-4a25-aefc-0567bf123abc",
    "email": "utente@example.com"
  }
  ```

### POST /api/v1/auth/login
- Request body schema:
  - email: string, formato email, obbligatorio.
  - password: string, obbligatorio.
- Response body schema:
  - 200 OK: { "token": string (JWT), "expires_in": integer (sec) }
  - 400 Bad Request: { "error": string }
  - 401 Unauthorized: { "error": "Invalid credentials" }
- Esempio request:
  ```json
  {
    "email": "utente@example.com",
    "password": "Pa$$w0rd123"
  }
  ```
- Esempio response 200:
  ```json
  {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...abc",
    "expires_in": 3600
  }
  ```

### POST /api/v1/auth/reset-password/request
- Request body schema:
  - email: string, formato email, obbligatorio.
- Response body schema:
  - 200 OK: { "message": "Reset link sent if email registered" }
  - 400 Bad Request: { "error": string }
- Esempio request:
  ```json
  {
    "email": "utente@example.com"
  }
  ```
- Esempio response 200:
  ```json
  {
    "message": "Reset link sent if email registered"
  }
  ```

### POST /api/v1/auth/reset-password/confirm
- Request body schema:
  - token: string, codice/token temporaneo ricevuto via email, obbligatorio.
  - new_password: string, nuova password conforme policy, obbligatorio.
- Response body schema:
  - 200 OK: { "message": "Password updated successfully" }
  - 400 Bad Request: { "error": string }
  - 401 Unauthorized: { "error": "Invalid or expired token" }
- Esempio request:
  ```json
  {
    "token": "abcdef1234567890",
    "new_password": "NewPa$$w0rd!"
  }
  ```
- Esempio response 200:
  ```json
  {
    "message": "Password updated successfully"
  }
  ```

### GET /api/v1/protected/resource (esempio endpoint protetto)
- Header:
  - Authorization: "Bearer {token}"
- Response body schema:
  - 200 OK: { "data": "protected info" }
  - 401 Unauthorized: { "error": "Invalid or expired token" }

## 6. Business logic
- Registrazione: verifica unicità email, validazione password, hashing sicuro con salt, persistenza utente.
- Login: verifica email e password hash, generazione JWT con firma HMAC o RSA, durata 1 ora.
- Reset password richiesta: verifica email registrata, generazione token reset temporaneo, invio email con link o codice.
- Reset conferma: validazione token reset, aggiornamento password con nuova hash.
- Middleware verifica token JWT per accesso risorse protette.
- Rate limiting per tentativi login e reset password (es. max 5 tentativi in 10 minuti).
- Logging di eventi login, reset, errori con timestamp e dati minimi per sicurezza.
- Validazione input su tutti campi per evitare injection o dati errati.
- Risposta coerente e standardizzata in JSON con codici HTTP appropriati.
- Concorrenza su registrazione email gestita tramite vincoli di unicità DB e gestione errori.

## 7. Persistenza e integrazioni
- PostgreSQL con due tabelle principali:
  - users: id UUID PK, email varchar unico, password_hash varchar, salt varchar, created_at timestamp, updated_at timestamp.
  - password_resets: id UUID PK, user_id FK, reset_token varchar univoco, expires_at timestamp, used boolean.
- Indici su email (unico) e reset_token (unico).
- Vincoli FK tra password_resets.user_id e users.id.
- Migrazioni gestite tramite tool tipo Flyway o Liquibase.
- Integrazione con SMTP o provider email esterno per invio reset.
- JWT firmati e validati con chiavi gestite in configurazione sicura.
- Database configurato per supportare alta concorrenza e scalabilità orizzontale.

## 8. Autenticazione e autorizzazione
- Autenticazione tramite email e password con validazione password hashata.
- Generazione e gestione token JWT firmati con scadenza temporale (1 ora).
- Middleware Bear per filtrare e autorizzare accesso a risorse protette verificando token.
- Nessun livello di autorizzazione differenziata o ruoli.
- Reset password con token temporaneo inviato via email per autorizzare modifica.
- Limitazione tentativi per mitigare attacchi brute force.

## 9. Gestione errori
- Errori input: risposta 400 con messaggio specifico (“Invalid email format”, “Password too weak”).
- Email già esistente in registrazione: 409 Conflict.
- Credenziali errate in login: 401 Unauthorized.
- Token JWT assente, invalido o scaduto: 401 Unauthorized.
- Token reset password scaduto o inválido: 401 Unauthorized.
- Errore interno server: 500 Internal Server Error con logging dettagliato.
- Risposte errori sempre JSON con struttura uniforme:
  ```json
  { "error": "descrizione" }
  ```
- Logging degli errori per monitoraggio.
- Fallback sui servizi esterni (email) con retry limitato.

## 10. Strategia di test backend

### Modulo registrazione
- test_register_user_success (unit): verifica creazione utente con email valida e password conforme.
- test_register_duplicate_email (integration): verifica errore 409 se email già esiste.
- test_register_invalid_email_format (unit): verifica validazione email fallita.
- test_register_weak_password (unit): verifica rifiuto password non conforme.

### Modulo login
- test_login_success (unit): login con credenziali corrette genera token JWT.
- test_login_wrong_password (unit): login con password errata restituisce 401.
- test_login_nonexistent_email (unit): login con email non presente restituisce 401.
- test_login_rate_limit (integration): verifica blocco dopo tentativi ripetuti.

### Modulo reset password
- test_reset_request_success (integration): invio richiesta reset con email valida.
- test_reset_request_unknown_email (integration): invio richiesta con email non registrata.
- test_reset_confirm_success (unit): cambio password con token valido.
- test_reset_confirm_invalid_token (unit): tentativo con token scaduto o errato.
- test_reset_confirm_weak_password (unit): tentativo di reset con password non conforme.

### Middleware JWT
- test_access_protected_endpoint_valid_token (integration): accesso con token valido.
- test_access_protected_endpoint_missing_token (integration): accesso senza token ritorna 401.
- test_access_protected_endpoint_expired_token (integration): ritorna 401.

## 11. Rischi tecnici
- Gestione race condition su registrazioni simultanee con stessa email, mitigata da vincolo DB e gestione errori.
- Limiti di tentativi login/reset parziali, necessaria eventuale evoluzione per protezione DoS.
- Assenza di meccanismo refresh token nell’MVP potrebbe impattare UX.
- Dipendenza da provider email per reset password, necessità fallback o retry.
- Compliance e sicurezza dei log devono essere valutati secondo normativa aziendale.
- Scalabilità e performance dipendono corretta configurazione Bear e DB cluster.
- Potenziali vulnerabilità injection bloccate con validazione e sanitizzazione stringhe.

## 12. Struttura file proposta
``` 
auth-backend/
  Application.java           # Entry point app — public static void main(String[] args)
  config/
    Settings.java           # Configurazione app e security — class Settings
  models/
    User.java               # Entity User — class User {UUID id, String email, String passwordHash, String salt}
    PasswordResetToken.java # Entity Reset Token — class PasswordResetToken {UUID id, User user, String token, Timestamp expiresAt, boolean used}
  repositories/
    UserRepository.java     # Interfaccia e implementazione CRUD su User — User create(User user), Optional<User> findByEmail(String email)
    PasswordResetRepository.java # CRUD reset token
  services/
    AuthService.java        # Logica auth — User register(), String login(), void resetRequest(), void resetConfirm()
    EmailService.java       # Interfaccia invio email — void sendResetEmail(String email, String token)
    PasswordHasher.java     # Funzioni hashing password con salt — String hash(String password, String salt)
    InputValidator.java     # Validazione input email/password
    RateLimiter.java        # Limitazione tentativi login/reset
  middleware/
    JwtMiddleware.java      # Middleware verifica token JWT — boolean validateToken(String token)
  controllers/
    AuthController.java     # Endpoint REST auth — register(), login(), resetRequest(), resetConfirm()
    ProtectedController.java# Endpoint protetto esempio — getProtectedResource()
  exceptions/
    ApiException.java       # Class eccezioni API personalizzate
  logging/
    LoggingService.java     # Logging eventi critici
  tests/
    AuthServiceTest.java    # Unit tests su AuthService
    AuthControllerTest.java # Integration tests endpoint auth
    JwtMiddlewareTest.java  # Unit test JWT validation
```

## 13. Piano di implementazione
1. Definire e migrare schema PostgreSQL per users e password_resets.
2. Implementare modello User e PasswordResetToken con validazione.
3. Sviluppare UserRepository e PasswordResetRepository per DB interaction.
4. Realizzare AuthService con registrazione e hashing password sicuro.
5. Implementare endpoint /register con validazione input e controllo unicità.
6. Implementare login con verifica password e generazione JWT.
7. Creare JwtMiddleware per verifica token e protezione API.
8. Implementare reset password: endpoint richiesta, generazione token, invio email.
9. Implementare conferma reset con aggiornamento password.
10. Sviluppare RateLimiter per protezione tentativi ripetuti.
11. Configurare sistema di logging sicuro per eventi critici.
12. Scrivere test unitari e di integrazione per ogni modulo.
13. Mettere in opera il monitoraggio base di log ed eventi.
14. Documentare API (request/response, errori, esempi) e configurazioni sicurezza.
15. Eseguire test di performance e sicurezza basici.
16. Preparare ambiente per deploy scalabile e stateless.