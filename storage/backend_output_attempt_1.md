MODULE: BACKEND VERSION: 1
## 1. Obiettivo backend
Progettare e implementare un backend API RESTful in Java, utilizzando il framework Bear, che offra un sistema di autenticazione utenti. Il sistema deve permettere la registrazione tramite email e password, il login con generazione di token JWT per sessioni stateless, la protezione degli endpoint tramite validazione JWT, un endpoint base per il reset password con invio di link temporaneo via email, garantendo elevati standard di sicurezza, scalabilità orizzontale, logging centralizzato degli eventi e la persistenza dati su PostgreSQL.

## 2. Assunzioni tecniche
- L’unico dato utente richiesto per registrazione è email (univoca) e password.
- La password è memorizzata in modo sicuro tramite hashing e salting.
- I token JWT sono firmati con chiave segreta e hanno durata fissa senza revoca.
- Comunicazioni sempre via HTTPS.
- Endpoint pubblici: registrazione, login, reset password.
- Tutti gli altri endpoint sono protetti da middleware Bear che verifica validità token JWT.
- Link per reset password è temporaneo, inviato via email senza rivelare esistenza email.
- Nessuna gestione di ruolo o permessi, social login, conferma email o reset avanzato.
- Errori esposti con messaggi generici per evitare leakage informativo.
- Logging centralizzato degli eventi autenticazione compatibile GDPR.
- Prevalente architettura stateless per consentire scalabilità orizzontale.
- Validazione input per prevenire injection e attacchi comuni.

## 3. Architettura backend
Architettura a microservizio API stateless basata su framework Bear con i seguenti componenti:
- Controller REST che espongono gli endpoint API
- Service layer che incapsula la business logic di autenticazione e gestione utenti
- Data Access Object (DAO) per interazione con PostgreSQL
- Middleware JWT Bear per la gestione di verifica token e protezione endpoint
- Modulo sicurezza per hashing password e generazione token JWT
- Modulo invio email per gestione link reset password
- Modulo logging centralizzato per eventi autenticazione
L’architettura è modulare, separando responsabilità, facilitando testabilità e manutenzione.

## 4. Moduli e responsabilità
- UserModel: definizione schema dati utente con vincoli (email unica)
- UserRepository: interfaccia DAO per CRUD utenti in PostgreSQL
- PasswordService: hashing e salting sicuro delle password
- JwtService: generazione, firma, verifica token JWT con scadenza configurabile
- AuthController: endpoint REST per registrazione, login e reset password pubblici
- ProtectedController (esempio): endpoint protetti accessibili solo con JWT valido
- JwtMiddleware: middleware Bear per verifica token JWT e autorizzazione
- EmailService: invio email per reset password con link temporaneo
- LoggingService: gestione logging centralizzato eventi autenticazione con anonimizzazione dati sensibili
- InputValidation: validazione base su email e password nelle richieste API
- Config: gestione configurazioni app (es. JWT secret, durata token, SMTP server)

## 5. API principali

### POST /api/v1/auth/register
- Request body schema:
  - email: string, formato email RFC valida, obbligatorio
  - password: string, minimo 8 caratteri, obbligatorio
- Response body schema:
  - 201 Created: { "message": "User registered successfully" }
  - 400 Bad Request: { "error": "Invalid email or password format" }
  - 409 Conflict: { "error": "Email already registered" }
- Esempio:
  - Request:
    ```json
    {
      "email": "user@example.com",
      "password": "StrongPassword123"
    }
    ```
  - Response 201:
    ```json
    { "message": "User registered successfully" }
    ```

### POST /api/v1/auth/login
- Request body schema:
  - email: string, formato email valida, obbligatorio
  - password: string, obbligatorio
- Response body schema:
  - 200 OK: { "token": "<JWT token>" }
  - 401 Unauthorized: { "error": "Invalid credentials" }
- Esempio:
  - Request:
    ```json
    {
      "email": "user@example.com",
      "password": "StrongPassword123"
    }
    ```
  - Response 200:
    ```json
    {
      "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
    }
    ```

### POST /api/v1/auth/reset-password
- Request body schema:
  - email: string, formato email valida, obbligatorio
- Response body schema:
  - 200 OK: { "message": "If the email is registered, a reset link has been sent" }
- Esempio:
  - Request:
    ```json
    {
      "email": "user@example.com"
    }
    ```
  - Response 200:
    ```json
    { "message": "If the email is registered, a reset link has been sent" }
    ```

### Esempio endpoint protetto
GET /api/v1/protected/resource
- Header: Authorization: Bearer <JWT>
- Response:
  - 200 OK: { "data": "protected data" }
  - 401 Unauthorized: { "error": "Invalid or missing token" }

## 6. Business logic
- Registrazione: verifica formato email, controllo univocità, hashing salting password, salvataggio in DB.
- Login: verifica email esistenza, confronto password hash, generazione token JWT firmato con payload minimo (es. userId, email, expiry).
- Reset password: generazione token reset temporaneo associato all’email, invio via email senza rivelare esistenza email per privacy.
- Middleware JWT: estrazione token da header Authorization, verifica firma, validità e scadenza token, abilitazione accesso a endpoint protetti.
- Logging: registrazione centralizzata di eventi (registrazione, login successo/fallito, reset password, errori), anonimizzando dati sensibili.

## 7. Persistenza e integrazioni
- Database PostgreSQL con tabella utenti:
  - campi principali: id (PK), email (unique), password_hash, salt, created_at, updated_at
- Integrazione SMTP (provider da configurare) per invio email con link reset password contenente token temporaneo.
- Configurazione tramite file/environment per segreti JWT, parametri DB, SMTP ecc.
- Logging centralizzato su sistema esterno o file con anonimizzazione.

## 8. Autenticazione e autorizzazione
- Autenticazione basata su token JWT firmato con chiave segreta configurata, durata fissa (es. 1 ora).
- Nessuna autorizzazione avanzata (ruoli/permessi) prevista.
- Middleware Bear per estrarre e validare token JWT e bloccare accesso endpoint protetti.
- Endpoint register, login e reset password accessibili senza autenticazione.

## 9. Gestione errori
- Messaggi di errore generici per non rivelare informazioni sensibili (es. login o reset password).
- Codici HTTP corretti per contesti di errore (400, 401, 403, 409).
- Validazione input con ritorno errori specifici ma non dettagli sicurezza.
- Logging errori e anomalie senza dati personali esposti.
- Trattamento eccezioni nel backend con messaggi anonimi verso client.

## 10. Strategia di test backend

### Modulo AuthController
- test_register_user_success (integration): verifica registrazione con dati validi ritorna 201 e utente creato
- test_register_duplicate_email (integration): verifica ritorna 409 se email già esiste
- test_register_invalid_email_format (unit): validazione email formale rifiuta input errati
- test_login_success (integration): login con credenziali corrette ritorna token JWT valido
- test_login_invalid_credentials (integration): login con password errata ritorna 401 generico
- test_reset_password_email_sent (integration): reset con email esistente invia email e ritorna messaggio generico
- test_reset_password_email_not_exist (integration): reset con email non registrata ritorna messaggio generico

### Modulo JwtMiddleware
- test_token_validation_success (unit): token valido autorizza accesso endpoint protetto
- test_token_expired (unit): token scaduto blocca accesso
- test_missing_token (unit): assenza token blocca accesso
- test_invalid_token_signature (unit): token firmato falsificato blocca accesso

### Modulo PasswordService
- test_password_hashing_and_verification (unit): verifica corretto hashing e confronto password

### Modulo EmailService
- test_email_send_success (integration): mock invio email con link reset password

## 11. Rischi tecnici
- Assenza meccanismi anti-brute force espone a tentativi di forza bruta su login.
- Mancata revoca token (logout remoto) può creare rischi di sicurezza se un token è compromesso.
- Sync temporale in ambienti distribuiti potrebbe invalidare token JWT.
- Dipendenza da provider SMTP esterno per invio email reset.
- Potenziali criticità nella gestione GDPR per logging centralizzato se dati non anonimizzati correttamente.
- Limitata complessità funzionale può rendere il sistema meno resistente ad attacchi e meno flessibile.

## 12. Struttura file proposta
``` 
project_root/
  Main.java                        # Entry point app — public static void main(String[] args)
  config/
    Settings.java                 # Configurazione app — class Settings { JWT secret, durations, smtp }
  models/
    User.java                    # Modello utente — class User { id, email, passwordHash, salt }
  repositories/
    UserRepository.java          # DAO utenti — interface UserRepository { findByEmail(), save() }
  services/
    PasswordService.java         # Hashing e verifica password — class PasswordService { hash(), verify() }
    JwtService.java              # Generazione e validazione JWT — class JwtService { generate(), validate() }
    EmailService.java            # Invio email reset — class EmailService { sendResetLink() }
    LoggingService.java          # Logging eventi autenticazione — class LoggingService { logEvent() }
  controllers/
    AuthController.java          # Endpoint auth — void register(), void login(), void resetPassword()
    ProtectedController.java     # Esempio endpoint protetto — void protectedResource()
  middleware/
    JwtMiddleware.java           # Middleware verifica token — void verifyToken()
  validators/
    InputValidator.java          # Validazioni input email, password — boolean isValidEmail(), isValidPassword()
  tests/
    AuthControllerTest.java      # Test auth controller — test_register_user_success(), test_login_invalid()
    JwtMiddlewareTest.java       # Test middleware token — test_token_validation_success(), test_token_expired()
    PasswordServiceTest.java     # Test hashing password — test_password_hashing_and_verification()
    EmailServiceTest.java        # Test invio email reset — test_email_send_success()
```

## 13. Piano di implementazione
1. Progettare e creare schema utenti PostgreSQL con vincolo unique su email.
2. Implementare PasswordService con hashing e salting sicuro.
3. Sviluppare UserRepository per accesso dati utenti.
4. Realizzare InputValidator per validazione email e password.
5. Implementare AuthController con endpoint /register e gestione errori.
6. Implementare endpoint /login con verifica credenziali e generazione JWT tramite JwtService.
7. Configurare JwtMiddleware per proteggere endpoint diversi da quelli pubblici.
8. Sviluppare EmailService per invio link reset password.
9. Implementare endpoint /reset-password con generazione link temporaneo e invio email.
10. Integrare LoggingService per logging centralizzato di eventi autenticazione.
11. Configurare HTTPS nell’ambiente di deploy.
12. Realizzare test unitari e di integrazione per tutti i moduli secondo strategia dedicata.
13. Effettuare test funzionali end-to-end dei flussi registrazione, login, accesso protetto e reset password.
14. Preparare la documentazione tecnica e istruzioni per deploy e configurazione.