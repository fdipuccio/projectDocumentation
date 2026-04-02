MODULE: BACKEND VERSION: 1
## 1. Obiettivo backend
Realizzare un backend modulare in Java con Bear framework per la gestione sicura dell’autenticazione utenti tramite REST API. Le funzionalità principali includono registrazione utente con validazione email e password, login con generazione token JWT valido per 1 ora, protezione di endpoint tramite verifica token JWT, gestione del reset password con generazione token temporaneo valido 15 minuti e invio email tramite sistema esterno. Il backend deve garantire performance rapide (<200ms per login/registrazione), sicurezza (hashing password con bcrypt, canali HTTPS) e scalabilità orizzontale. 

## 2. Assunzioni tecniche
- Password devono rispettare policy minima 8 caratteri con lettere e numeri.
- Token JWT ha durata 1 ora.
- Token reset password ha durata 15 minuti.
- Invio email reset password tramite sistema esterno integrato.
- Bear framework supporta middleware o filtro per verifica token JWT.
- PostgreSQL usato per persistenza utenti, token reset e logging eventi critici.
- Comunicazione client-server tramite HTTPS.
- Hashing e salting delle password con bcrypt.
- Blocchi tentativi login falliti e mitigazioni CSRF implementate a livello applicativo.
- Logging eventi critici di autenticazione e reset password.
- Mancano funzionalità di logout/revoca token o multi-fattore/autorizzazioni avanzate.
- API progettate per semplicità e basso coupling, compatibili con ambiente enterprise.

## 3. Architettura backend
Architettura modulare a livelli:
- Layer Presentazione: REST API esposte usando Bear framework.
- Layer Business Logic: servizi che gestiscono registrazione, login, reset password, validazione token.
- Layer Persistenza: DAO e repository per accesso a PostgreSQL (utenze, token reset, log).
- Layer Sicurezza: componente per hashing con bcrypt, generazione/verifica JWT, filtro Bear per protezione endpoint.
- Integrazione esterna per invio email reset password.
- Logging centralizzato per eventi critici.
Il backend sarà implementato come API stateless per scalabilità orizzontale.

## 4. Moduli e responsabilità
- UserModule: gestione utenti, registrazione, validazione password/email, hashing password.
- AuthModule: gestione login, generazione e validazione token JWT.
- ResetPasswordModule: gestione richiesta reset, generazione token reset, convalida token, cambio password.
- SecurityModule: filtro Bear per protezione endpoint via JWT, gestione blocco tentativi login falliti.
- EmailService: invio email reset password tramite sistema esterno.
- PersistenceModule: DAO per utenti, token reset e log eventi.
- LoggingModule: audit eventi di login, reset password, errori di autenticazione.

## 5. API principali
- POST /api/v1/auth/register
  - Request body: 
    ```json
    {
      "email": "string, valid email, unique, required",
      "password": "string, min 8 characters, letters and numbers, required"
    }
    ```
  - Response body:
    - 201 Created
    ```json
    {
      "message": "User registered successfully"
    }
    ```
    - 400 Bad Request (validation errors)
    - 409 Conflict (email già registrata)
  - Esempio:
    Request:
    ```json
    {
      "email": "utente@example.com",
      "password": "Password123"
    }
    ```
    Response 201:
    ```json
    {
      "message": "User registered successfully"
    }
    ```

- POST /api/v1/auth/login
  - Request body:
    ```json
    {
      "email": "string, required",
      "password": "string, required"
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
    - 401 Unauthorized (credenziali errate o account bloccato)
  - Esempio:
    Request:
    ```json
    {
      "email": "utente@example.com",
      "password": "Password123"
    }
    ```
    Response 200:
    ```json
    {
      "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "expires_in": 3600
    }
    ```

- POST /api/v1/auth/reset-password/request
  - Request body:
    ```json
    {
      "email": "string, required"
    }
    ```
  - Response body:
    - 200 OK
    ```json
    {
      "message": "Password reset email sent if user exists"
    }
    ```
  - Funzione: genera token reset valido 15 minuti, memorizza, invia email tramite sistema esterno.
  - Esempio:
    Request:
    ```json
    {
      "email": "utente@example.com"
    }
    ```
    Response 200:
    ```json
    {
      "message": "Password reset email sent if user exists"
    }
    ```

- POST /api/v1/auth/reset-password/confirm
  - Request body:
    ```json
    {
      "token": "string, required",
      "new_password": "string, min 8 characters, letters and numbers, required"
    }
    ```
  - Response body:
    - 200 OK
    ```json
    {
      "message": "Password updated successfully"
    }
    ```
    - 400 Bad Request (token scaduto, invalido, password non conforme)
  - Esempio:
    Request:
    ```json
    {
      "token": "reset_token_string",
      "new_password": "NewPassword123"
    }
    ```
    Response 200:
    ```json
    {
      "message": "Password updated successfully"
    }
    ```

- Middleware Bear framework per la protezione endpoint:
  - Controlla presenza token JWT valido su header Authorization: Bearer <token>.
  - Rifiuta richieste con 401 Unauthorized in caso di assenza o token invalido/scaduto.

## 6. Business logic
- Registrazione: verifica che email sia univoca, valida e password rispetti policy; esegue hashing e salting con bcrypt prima di salvare utente su DB.
- Login: verifica corrispondenza email e password tramite hash bcrypt; se valido genera JWT con durata 1 ora.
- Reset password richiesta: genera token random associato a utente con scadenza 15 minuti, salva su DB, invia email con link/token.
- Reset password conferma: verifica token reset valido, aggiorna password nuova con hashing bcrypt, invalida token.
- Sistema blocco tentativi login falliti per proteggere da attacchi brute force.
- Logging eventi critici (successi o tentativi falliti di login e reset).

## 7. Persistenza e integrazioni
- PostgreSQL schema:
  - Users table: id (PK), email (unique), password_hash, created_at, updated_at, login_fail_count, locked_until.
  - ResetTokens table: id, user_id (FK), token, expiration_time, created_at.
  - AuditLog table: id, user_id, event_type (login_success, login_fail, reset_request, reset_success, reset_fail), timestamp, ip_address, details.
- Integrazione esterna per invio email (API HTTP esterna).
- Connessione DB ottimizzata per performance e scalabilità.
- Transaction management su operazioni critiche.

## 8. Autenticazione e autorizzazione
- JWT con algoritmo HS256 o equivalente, scadenza 1 ora (configurabile).
- Bear framework usato per filtro middleware che verifica token JWT su chiamate protette.
- No ruoli o livelli di permessi: autenticazione semplice basata su validità token.
- Blocchi temporanei di account dopo N tentativi login falliti consecutivi (configurabile).
- Trasmissione dati sulla rete solo tramite HTTPS.

## 9. Gestione errori
- Risposte HTTP con codici appropriati (400, 401, 409, 500).
- Messaggi di errore chiari sugli input non validi (email, password, token scaduto).
- Gestione token reset scaduto o manomesso con risposta 400.
- Blocco temporaneo in caso di troppi tentativi login falliti, con messaggio esplicativo.
- Logging errori interni e di sicurezza.
- Fallback generico 500 per errori inattesi.

## 10. Strategia di test backend
- test_register_user_success (integration)
  - Verifica registrazione utente con email e password valide genera 201.
- test_register_user_duplicate_email (integration)
  - Verifica registrazione con email già presa restituisce 409.
- test_register_invalid_password (unit)
  - Verifica fallimento registrazione con password non conforme.
- test_login_success (integration)
  - Verifica login corretto restituisce token JWT valido.
- test_login_invalid_credentials (integration)
  - Verifica login con credenziali errate restituisce 401.
- test_login_account_locked (integration)
  - Verifica blocco dopo N tentativi falliti.
- test_reset_password_request_existing_email (integration)
  - Verifica token reset creato e email inviata se email esistente.
- test_reset_password_request_non_existing_email (integration)
  - Verifica risposta 200 indistinta per email non esistente.
- test_reset_password_confirm_valid_token (integration)
  - Verifica cambio password con token valido.
- test_reset_password_confirm_invalid_token (integration)
  - Verifica errore con token scaduto/invalido.
- test_protected_endpoint_no_token (integration)
  - Verifica richiesta senza token su endpoint protetto restituisce 401.
- test_protected_endpoint_invalid_token (integration)
  - Verifica richiesta con token invalido restituisce 401.
- test_password_hashing (unit)
  - Verifica che password non venga mai salvata in chiaro.
- performance_login_registration (e2e)
  - Verifica tempi risposta login/registrazione < 200ms.
- test_error_handling_edge_cases (integration)
  - Verifica gestione errori vari come token malformato, DB non disponibile.

## 11. Rischi tecnici
- Dipendenza da sistema esterno email può introdurre ritardi o vulnerabilità.
- Mancanza gestione esplicita revoca token JWT può causare rischi sicurezza.
- Blocchi tentativi login necessitano tuning per evitare denial of service involontari.
- Potenziali attacchi replay token JWT da mitigare in futuro.
- Ambiguità futura su ruoli autorizzativi non ancora gestiti.
- Monitoraggio/logging limitato potrebbe non coprire casi di produzione più complessi.
- Policy password attuale basica, possibile necessità di revisione.

## 12. Struttura file proposta
``` 
auth-backend/
  Main.java                  # Entry point app — public static void main(String[] args)
  config/
    Settings.java            # Configurazione app (DB, JWT secret, email API) — class Settings
  models/
    User.java                # Modello User — class User {id, email, passwordHash,...}
    ResetToken.java          # Modello token reset — class ResetToken {id, userId, token, expiration,...}
    AuditLog.java            # Modello audit log — class AuditLog {id, userId, eventType,...}
  repositories/
    UserRepository.java      # Accesso DB users — interface con metodi CRUD
    ResetTokenRepository.java# Accesso DB token reset
    AuditLogRepository.java  # Accesso DB audit log
  services/
    UserService.java         # Logica gestione utenti, registrazione, hashing — public void register(...)
    AuthService.java         # Login, JWT generazione/validazione — public String login(...)
    ResetPasswordService.java# Gestione token reset, cambio password — public void requestReset(...)
    EmailService.java        # Invio email tramite sistema esterno — public void sendResetEmail(...)
    SecurityService.java     # Gestione sicurezza (hashing, blocco login, filtraggio JWT)
  controllers/
    AuthController.java      # Endpoint REST auth (register, login, reset password) — @Path /api/v1/auth
  filters/
    JwtAuthFilter.java       # Filtro Bear per verifica token JWT — implements BearFilter
  exceptions/
    CustomException.java     # Definizione eccezioni custom (es validation errors, token expired)
  utils/
    PasswordUtils.java       # Funzioni hashing/salting password con bcrypt
    JwtUtils.java            # Funzioni gestione creazione/verifica JWT
  logs/
    # Cartella per file di log di audit
  tests/
    UserServiceTest.java     # Test unit registro utenti e validazioni
    AuthServiceTest.java     # Test login e JWT
    ResetPasswordServiceTest.java # Test reset password, token
    AuthControllerTest.java  # Test integration controller endpoint
    PerformanceTest.java     # Test prestazioni login/registrazione
```

## 13. Piano di implementazione
1. Definire schema dati utenti, token reset e log eventi in PostgreSQL.
2. Implementare modelli e repository per utenti, token reset, audit log.
3. Implementare UserService con registrazione, validazioni, hashing password.
4. Implementare AuthService con login, verifica password, generazione JWT.
5. Configurare Bear framework: definire JwtAuthFilter per proteggere API.
6. Implementare AuthController con endpoint register, login protetti da validazioni.
7. Implementare ResetPasswordService, EmailService per generazione token reset e invio email.
8. Esporre endpoint reset-password/request e reset-password/confirm in AuthController.
9. Implementare logging audit eventi critici (login, reset, errori).
10. Gestire errori e edge case (tentativi login falliti, token scaduti).
11. Configurare HTTPS per comunicazioni sicure.
12. Realizzare test unitari, integration e di performance per garantire copertura e SLA.
13. Effettuare revisione codici, security audit e verifiche finali prima del rilascio.