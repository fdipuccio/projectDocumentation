MODULE: BACKEND VERSION: 1
## 1. Obiettivo backend
Sviluppare un backend Java basato su Bear framework per fornire un sistema sicuro e modulare di autenticazione utenti con funzionalità di registrazione, login tramite email e password, gestione token JWT per accesso protetto e flusso di reset password via email.

## 2. Assunzioni tecniche
- Validazione email formattata correttamente e password con forza minima in fase di registrazione e login.
- Password salvate in forma hashed con bcrypt o equivalente.
- JWT firmati con chiave sicura e validità configurabile tra 15-60 minuti.
- Tutti gli endpoint eccetto /register e /login sono protetti da verifica token JWT.
- Logging strutturato per monitoraggio accessi, errori e operazioni critiche.
- Gestione errori centralizzata con risposte standard e latenza accettabile.
- Limitazione tentativi login per mitigare brute force.
- Dipendenza da infrastruttura di invio email per reset password con necessità di fallback/ retry.

## 3. Architettura backend
Modulare in livelli:
- Controller: gestisce richieste HTTP e risposte.
- Service: logica di business, validazione, gestione token, hashing password.
- Repository: accesso e manipolazione dati su PostgreSQL.
- Security: gestione JWT, middleware/interceptor Bear per protezione endpoint.
- Integration: invio email reset password con gestione fallback e retry.
- Logging: registrazione eventi sicurezza e sistema.

## 4. Moduli e responsabilità
- UserRepository: operazioni CRUD su utenti, controllo unicità email.
- AuthController: endpoint /register, /login, /reset-password/request, /reset-password/confirm.
- AuthService: logica registrazione, login, gestione token, limitazione tentativi.
- JwtProvider: generazione e validazione JWT.
- PasswordHasher: hashing e verifica password.
- EmailService: invio email con token reset e gestione retry/fallback.
- LoggingService: log strutturato degli eventi.
- JwtAuthMiddleware: interceptor per protezione API protette.

## 5. API principali
- POST /register
  Request: {email: string, password: string} (email formato valido, password min 8 char)
  Response: 201 {message: "User registered"}, 400 validazione, 409 email esistente
  Esempio: {"email":"user@example.com", "password":"StrongPass123"}

- POST /login
  Request: {email: string, password: string}
  Response: 200 {token: string JWT}, 400 errori validazione, 401 credenziali errate, 429 troppi tentativi
  Esempio: {"email":"user@example.com", "password":"StrongPass123"}

- POST /reset-password/request
  Request: {email: string}
  Response: 200 {message: "Reset token sent if email exists"}, 400 validazione
  Esempio: {"email":"user@example.com"}

- POST /reset-password/confirm
  Request: {token: string, newPassword: string}
  Response: 200 {message: "Password updated"}, 400 validazione, 401 token invalido/scaduto
  Esempio: {"token":"reset-token-uuid", "newPassword":"NewStrongPass123"}

## 6. Business logic
- Registrazione con controllo email unica, validazione input, hashing password.
- Login con controllo credenziali, gestione blocco tentativi falliti.
- Generazione token JWT con payload utente e durata configurabile.
- Middleware che valida JWT su API protette, blocca richieste non autorizzate.
- Reset password con generazione token univoco, invio email e validazione token in conferma.
- Logging di tutti i passaggi critici: accessi, errori, modifiche password.
- Gestione errori e fallimenti DB con risposte standard, retry e fallback per email.

## 7. Persistenza e integrazioni
- PostgreSQL, tabella utenti con colonne: id (PK), email (unique), password_hash, reset_token, timestamp vari.
- Indice unique sulla email.
- Gestione transazioni per evitare race condition su reset token.
- Integrazione con servizio email esterno, con retry e fallback configurabili.

## 8. Autenticazione e autorizzazione
- JWT firmati con chiave segreta, durata configurabile.
- Middleware Bear per validazione token JWT su endpoint protetti.
- Nessun ruolo utente multiplo o permessi avanzati (out of scope).
- Limitazione tentativi login per prevenire attacchi brute force.

## 9. Gestione errori
- Risposte uniformi in JSON con codici HTTP standard: 2xx, 4xx, 5xx.
- Messaggi di errore generici per sicurezza (es: in login non rivelare se email o password errate).
- Gestione errori DB con risposte 500 e log dettagliati.
- Timeout e fallback su email service.

## 10. Strategia di test backend
- test_register_user_success (unit/integration): verifica registrazione valida con nuova email.
- test_register_duplicate_email (unit/integration): verifica errore su email già registrata.
- test_login_success (unit/integration): verifica login corretto e token JWT.
- test_login_fail_block_after_n_attempts (integration): verifica blocco dopo tentativi errati.
- test_reset_password_request_and_confirm (integration/e2e): verifica flusso completo reset password.
- test_jwt_validation_middleware (unit): verifica blocco accesso senza/ con token invalidi.
- test_db_failure_handling (integration): verifica comportamento sistema a failure database.
- test_email_service_retry_fallback (integration): verifica retry e fallback su servizio email.

## 11. Rischi tecnici
- Dipendenza da servizio email esterno, necessaria strategia retry/fallback.
- Possibili race condition su richieste reset password, gestite da transazioni e locking DB.
- Limitata gestione revoca token JWT, solo scadenza temporale.
- Mancanza verifica email esplicita può favorire account fraudolenti.
- Necessità affinare strategie anti brute force oltre blocco base.
- Ambiguità ruoli e protezione endpoint da chiarire per future estensioni.

## 12. Struttura file proposta
``` 
project_name/
  Main.java               # Entry point applicazione — public class Main { public static void main(String[] args); }
  Config.java             # Configurazione app — public class Config { public String jwtSecret; public int jwtExpiration; etc. }
  models/
    User.java             # Modello User — class User { Long id; String email; String passwordHash; String resetToken; Timestamp fields; }
  repositories/
    UserRepository.java   # Accesso dati — interface UserRepository { Optional<User> findByEmail(String); void save(User); }
  services/
    AuthService.java      # Logica autenticazione — register(), login(), resetPasswordRequest(), resetPasswordConfirm(), limitLoginAttempts()
    JwtProvider.java      # Generazione e validazione JWT — generateToken(User), validateToken(String)
    PasswordHasher.java   # Password hashing — hashPassword(String), verifyPassword(String, String)
    EmailService.java     # Invio email con retry/fallback — sendResetTokenEmail(String, String)
    LoggingService.java   # Logging strutturato — logAccess(), logError(), logSecurityEvent()
  controllers/
    AuthController.java   # Endpoint REST API — register(), login(), resetPasswordRequest(), resetPasswordConfirm()
  security/
    JwtAuthMiddleware.java # Middleware Bear per verifica token — handle(Request)
  tests/
    AuthServiceTest.java  # Test unit/integration — testRegisterUserSuccess(), testLoginFailBlockAfterAttempts(), testResetPasswordFlow()
    JwtProviderTest.java  # Test unit — testGenerateAndValidateToken()
    EmailServiceTest.java # Test integration — testEmailSendRetryFallback()
```

## 13. Piano di implementazione
1. Definire schema DB e migrazioni con tabella utenti e vincoli.
2. Implementare UserRepository con operazioni base e controllo unicità.
3. Sviluppare PasswordHasher con bcrypt.
4. Realizzare JwtProvider per generazione/validazione JWT.
5. Costruire AuthService con logica registrazione, login, token, limitazione tentativi.
6. Creare EmailService per invio email reset con retry e fallback.
7. Implementare AuthController con tutti gli endpoint REST.
8. Integrare JwtAuthMiddleware per protezione endpoint.
9. Implementare LoggingService e inserire log dettagliati nei moduli.
10. Scrivere e automatizzare test unitari, di integrazione e e2e per coprire happy path ed errori.
11. Effettuare test di carico e performance.
12. Documentare dettagli tecnici, incluse strategie di gestione concorrente reset password e fallback email.
13. Rilascio MVP con monitoraggio e review post deploy.