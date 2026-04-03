MODULE: BACKEND VERSION: 1
## 1. Obiettivo backend
Realizzare un backend di autenticazione utenti sicuro, performante e scalabile su stack Java, REST API, Bear framework, PostgreSQL e JWT. Il backend deve supportare registrazione utenti con email unica e password protetta, login con generazione di token JWT firmati e scaduti, protezione di endpoint tramite middleware di verifica token, reset password con meccanismo di invio codice/link temporaneo via email, persistenza dati utenti e sessioni in PostgreSQL, logging sicuro e monitoraggio eventi critici. È previsto un sistema backend only, con API REST documentate e gestione errori consistente.

## 2. Assunzioni tecniche
- Email validata per unicità ma senza conferma via link di attivazione nell’MVP.
- Password con minimo 8 caratteri, inclusi lettere, numeri e simboli, validate lato backend.
- Password salvate hashed con salt sicuro, preferibilmente algoritmo bcrypt o equivalente.
- Token JWT con firma sicura (es. HS256 o RS256) e scadenza impostata intorno a 1 ora.
- Invio reset password tramite codice o link temporaneo via email (con sistema esterno SMTP configurato).
- Limiti di tentativi login/reset per mitigare attacchi brute force implementati a livello logico.
- Scalabilità orizzontale garantita dal disaccoppiamento stateless dei servizi e persistenza centralizzata su PostgreSQL.
- Logging centralizzato e sicuro con log di eventi critici (login, reset, errori).
- Input validation rigorosa per prevenire injection e vulnerabilità comuni.
- Middleware Bear per protezione endpoint con verifica token JWT.
- Nessun frontend o interfacce UI previste.
- Documentazione API REST conforme agli standard Bear e JSON schema per request/response.

## 3. Architettura backend
Architettura a moduli separati che gestisce richiesta HTTP tramite Bear framework, con controller REST per i diversi endpoint (registrazione, login, reset password), servizio di autenticazione per gestione logica token e credenziali, repository per accesso e persistenza database PostgreSQL, e middleware per verifica token JWT. Logging e monitoraggio sono integrati trasversalmente. Il sistema è disaccoppiato e stateless per scalabilità orizzontale, con database come unica fonte di verità per utenti e sessioni.

Flusso tipico:
- Client invia richiesta REST → Dispatcher Bear → Controller (es. AuthController)
- Controller esegue validazioni → richiama service autenticazione
- Service usa repository per lettura/scrittura utenti/tokens
- JWT generati validati da middleware Bear per accesso endpoint protetti
- Eventi critici loggati
- Risposta HTTP inviata client

## 4. Moduli e responsabilità
- Modello Dati (models): definizione User, PasswordResetToken, JWT Claims.
- Repository (repository): interfaccia con DB PostgreSQL per CRUD utenti e token.
- Servizi di Business (services): logica registrazione, login, generazione/verifica JWT, reset password.
- Controller (controllers): implementazione endpoint REST per registrazione, login, reset password.
- Middleware (middleware): filtro Bear per validazione token JWT su endpoint protetti.
- Configurazione (config): gestione parametri sicurezza, DB, JWT, email SMTP.
- Logging e Monitoraggio (logging): gestione logging sicuro eventi critici.
- Validazione input (validation): regole validazione campi input utente.
- Test (tests): unit, integration e functional test per tutti i moduli.

## 5. API principali

### POST /api/v1/auth/register
- Request body:
  ```json
  {
    "email": "string, email valida, unica",
    "password": "string, min 8 caratteri, lettere/numeri/simboli"
  }
  ```
- Response body:
  - 201 Created
  ```json
  {
    "userId": "UUID",
    "email": "string"
  }
  ```
  - 400 Bad Request: errori validazione
  - 409 Conflict: email già esistente

- Esempio:
  Request:
  ```json
  {
    "email": "utente@example.com",
    "password": "Password123!"
  }
  ```
  Response:
  ```json
  {
    "userId": "123e4567-e89b-12d3-a456-426614174000",
    "email": "utente@example.com"
  }
  ```

### POST /api/v1/auth/login
- Request body:
  ```json
  {
    "email": "string, email registrata",
    "password": "string"
  }
  ```
- Response body:
  - 200 OK
  ```json
  {
    "token": "string, JWT firmato",
    "expiresIn": 3600
  }
  ```
  - 400 Bad Request: errori validazione
  - 401 Unauthorized: credenziali errate

- Esempio:
  Request:
  ```json
  {
    "email": "utente@example.com",
    "password": "Password123!"
  }
  ```
  Response:
  ```json
  {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expiresIn": 3600
  }
  ```

### GET /api/v1/auth/protected-resource (esempio endpoint protetto)
- Headers: `Authorization: Bearer <token_jwt>`
- Response:
  - 200 OK con risorsa protetta
  - 401 Unauthorized se token mancante, invalido o scaduto

### POST /api/v1/auth/reset-password-request
- Request body:
  ```json
  {
    "email": "string, email registrata"
  }
  ```
- Response:
  - 200 OK: link/codice inviato (non indicare se email esiste per sicurezza)
  - 400 Bad Request: input errato

### POST /api/v1/auth/reset-password
- Request body:
  ```json
  {
    "resetToken": "string, temporaneo",
    "newPassword": "string, conforme policy password"
  }
  ```
- Response:
  - 200 OK password modificata
  - 400 Bad Request o 401 Unauthorized se token invalido o scaduto

## 6. Business logic
- Registrazione: validazione formato email e password, controllo unicità email, hashing password con salt sicuro, creazione record utente.
- Login: verifica esistenza email e confronto password hashed, generazione JWT con claims essenziali e firma, durata esplicita nel token.
- Middleware: estrazione token da header, verifica firma e scadenza, propagation errori 401 se invalidi.
- Reset password: generazione token temporaneo unico associato all’utente, invio email con link codice, validazione token e aggiornamento nuova password hashed.
- Limitazione tentativi: conteggio e temporizzazione per bloccare richieste e tentativi eccessivi.

## 7. Persistenza e integrazioni
- PostgreSQL con tabelle utenti, password reset token.
- Colonne utenti: id UUID PK, email unico, password_hash, salt, created_at, updated_at.
- Colonne reset token: id, user_id FK, token univoco, expiration timestamp.
- Integrazione SMTP per invio email reset password.
- Connessioni DB con pool configurato per scalabilità.
- Bear framework per REST e middleware.
  
## 8. Autenticazione e autorizzazione
- Autenticazione con email e password.
- JWT come meccanismo stateless di autenticazione per accesso protetto.
- Middleware Bear verifica token JWT firma, integrità, scadenza.
- Nessuna autorizzazione granulare o ruoli utente.
- Sessioni multiple consentite senza limitazioni specifiche.
- Protezione input e header API.

## 9. Gestione errori
- Risposte standardizzate JSON con codice errore e messaggio.
- 400 Bad Request per input invalidi con dettagli campo.
- 401 Unauthorized per token mancanti, invalidi, scaduti.
- 404 Not Found per risorse inesistenti in reset password o simili.
- 409 Conflict per email duplicata in registrazione.
- Log errori dettagliati con contesto per troubleshooting.
- Messaggi errori generici per non rivelare dati sensibili.

## 10. Strategia di test backend

### Modulo Registrazione
- test_register_user_success (integration): verifica registrazione valida crea utente; input email/password valide, expected 201 con userId.
- test_register_email_duplicate (integration): verifica errore email duplicata; input email già usata, expected 409.
- test_register_password_policy_fail (unit): verifica rifiuto password non conforme; input password corta, expected 400.

### Modulo Login
- test_login_success (integration): login con credenziali corrette; input email/password corrette, expected 200 con JWT.
- test_login_wrong_password (integration): login con password errata; input valido ma password sbagliata, expected 401.
- test_login_limit_brute_force (integration): verifica blocco dopo tentativi ripetuti; input sequenze errate, expected blocco temporaneo.

### Modulo Middleware JWT
- test_jwt_valid_token (unit): verifica token valido permette accesso; input token valido, expected pass-through.
- test_jwt_expired_token (unit): verifica token scaduto blocca accesso; input token con scadenza passata, expected 401.

### Modulo Reset Password
- test_reset_password_request_email_exists (integration): richiedi reset con email valida, expected 200 e codice inviato.
- test_reset_password_request_email_not_exists (integration): richiesta con email non esistente, expected 200 (non rivelare).
- test_reset_password_success (integration): modifica password con token valido; input token, nuova password conforme, expected 200.
- test_reset_password_invalid_token (integration): input token errato, expected 401.

## 11. Rischi tecnici
- Mancata conferma email può causare utenti non verificati.
- Politiche password e limiti tentativi da dettagliarsi per evitare vulnerabilità brute force o DoS.
- Gestione concorrenza registrazione email duplicata da gestire a livello DB con constraint unici e gestione eccezioni.
- Monitoraggio e logging devono garantire compliance sicurezza senza impattare performance.
- Mancanza autorizzazioni granulari limita estensioni future.
- Assenza meccanismo refresh token limitante per usabilità a lungo termine.
- Reset password dipende da affidabilità sistema email esterno.
- Potenziali attacchi DoS se input validation o limiti non sufficienti.

## 12. Struttura file proposta
``` 
authentication-backend/
  src/
    main/
      java/
        com/
          example/
            auth/
              Application.java                        # Entry point Bear app — public static void main(String[] args)
              config/
                AppConfig.java                       # Configurazioni app — class AppConfig
                JwtConfig.java                       # Parametri JWT — class JwtConfig
                EmailConfig.java                     # Config SMTP — class EmailConfig
              controllers/
                AuthController.java                  # REST endpoint auth — register(), login(), resetPasswordRequest(), resetPassword()
              middleware/
                JwtAuthMiddleware.java               # Bear middleware verifica JWT — boolean authenticate(Request req)
              models/
                User.java                           # Entity User — UUID id, String email, String passwordHash, String salt
                PasswordResetToken.java             # Entity reset token — UUID id, UUID userId, String token, Timestamp expiration
                JwtClaims.java                      # JWT claims model
              repository/
                UserRepository.java                  # Interfaccia DB utenti — User findByEmail(String email), void save(User user)
                PasswordResetTokenRepository.java   # DB reset token — PasswordResetToken findByToken(String token), void save(PasswordResetToken token)
              services/
                AuthService.java                    # Logica business auth — User register(), String login(), void resetPasswordRequest(), void resetPassword()
              utils/
                PasswordHasher.java                 # Hashing password + salt — String hash(String password, String salt)
                JwtTokenUtil.java                   # JWT creazione e verifica — String generateToken(User user), Claims validateToken(String token)
                EmailSender.java                    # Invio email — void sendResetLink(String email, String token)
              validation/
                InputValidator.java                 # Regole validazione input — boolean isValidEmail(), boolean isValidPassword()
              logging/
                AuditLogger.java                    # Log eventi critici — void logLogin(), logError()
    test/
      java/
        com/
          example/
            auth/
              AuthControllerTest.java               # Test integration controller endpoint auth
              AuthServiceTest.java                  # Test unit per logica AuthService
              JwtAuthMiddlewareTest.java            # Test middleware JWT validazione
              InputValidatorTest.java               # Test validazione input
```

## 13. Piano di implementazione
1. Progettare e creare schema DB utenti e password reset token in PostgreSQL con vincoli di unicità e indici.
2. Implementare modello dati, repository e configurazioni DB.
3. Implementare logica hashing password con salt in PasswordHasher.
4. Sviluppare endpoint registrazione con validazione input, controllo unicità email, hashing password e persistenza utente.
5. Implementare login con verifica credenziali e generazione token JWT firmato con scadenza.
6. Sviluppare middleware Bear per verifica token JWT valido su endpoint protetti.
7. Implementare reset password: generazione token temporaneo, salvataggio, invio email tramite SMTP, endpoint per aggiornamento password.
8. Configurare logging sicuro per eventi critici come login, reset password e errori.
9. Inserire validazione rigorosa per tutti input API per prevenire injection o dati sporchi.
10. Scrivere test unitari e di integrazione per tutti moduli e scenari normali/limite.
11. Predisporre monitoraggio logging e alert base per tentativi anomali o malfunzionamenti.
12. Redigere documentazione tecnica API REST, configurazioni di sicurezza e guide di implementazione/deployment.