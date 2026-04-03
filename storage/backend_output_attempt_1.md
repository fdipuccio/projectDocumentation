MODULE: BACKEND VERSION: 1
## 1. Obiettivo backend
Sviluppare un backend REST API in Java che gestisca l’autenticazione utenti con registrazione, login, generazione e validazione di token JWT, protezione degli endpoint tramite token e flusso di reset password. Il sistema deve garantire sicurezza degli accessi, logging strutturato, conformità GDPR, alta performance (risposta < 500 ms) e resilienza in caso di perdita temporanea del database. Non sono previsti frontend né gestione ruoli complessi.

## 2. Assunzioni tecniche
- Lato backend solo API REST su HTTPS sviluppate in Java con framework Bear.
- Persistenza dati su database PostgreSQL con schema ottimizzato per email unica e password hashata.
- Token JWT generati con durata configurabile da 15 a 60 minuti, default 30 minuti.
- Payload JWT minimale contenente userId.
- Password salvate tramite bcrypt (hashing e salting).
- Soglia di blocco login e reset password dopo 5 tentativi falliti con blocco di 15 minuti.
- Nessun meccanismo di refresh token o conferma email post-registrazione.
- Logging strutturato per eventi di autenticazione, errori e accessi critici.
- Monitoraggio tentativi sospetti e gestione errori di input e token.
- Nessun ruolo o autorizzazione avanzata è gestita.

## 3. Architettura backend
Architettura REST API monolitica modulare:
- Controller REST che espone endpoint.
- Service layer che implementa la business logic.
- Repository layer per accesso e manipolazione dati PostgreSQL tramite ORM o query sql.
- Componenti Middleware per validazione JWT e gestione limitazione tentativi.
- Componente Email service per l’invio del token reset password.
- Sistema di logging strutturato centralizzato.
- Configurazione centralizzata per setting sicurezza, durata token, limiti tentativi.
- Meccanismi di fallback e retry per resilienza in caso di perdita temporanea database.

## 4. Moduli e responsabilità
- UserModule: gestione utenti, registrazione, validazione email e password.
- AuthModule: login, generazione/validazione token JWT, limitazione tentativi login.
- ResetPasswordModule: gestione flusso reset password, invio token via email, validazione token.
- SecurityMiddleware: validazione token JWT sugli endpoint protetti, controllo blocco tentativi.
- PersistenceModule: interazione con PostgreSQL, modelli dati utente.
- LoggerModule: logging strutturato di eventi e errori.
- ConfigModule: gestione configurazioni runtime (token duration, soglie tentativi).
- EmailModule: invio email con token reset password.
- MonitoringModule: monitoraggio eventi sospetti e logging esteso.
- ErrorHandlingModule: gestione centralizzata delle eccezioni e risposte errore.

## 5. API principali

### POST /api/v1/auth/register
- Request body:
  - email: string, email valida e unica
  - password: string, min 8 caratteri, almeno lettere e numeri
- Response:
  - 201 Created
  - body { userId: string }
  - 400 Bad Request (validation error or email già registrata)
- Esempio:
  Request:
  ```json
  {
    "email": "utente@example.com",
    "password": "Pass1234"
  }
  ```
  Response 201:
  ```json
  {
    "userId": "uuid-utente"
  }
  ```

### POST /api/v1/auth/login
- Request body:
  - email: string
  - password: string
- Response:
  - 200 OK
  - body { token: string, expiresIn: int(minuti) }
  - 401 Unauthorized (credenziali errate, blocco tentativi)
- Esempio:
  Request:
  ```json
  {
    "email": "utente@example.com",
    "password": "Pass1234"
  }
  ```
  Response 200:
  ```json
  {
    "token": "jwt-token-string",
    "expiresIn": 30
  }
  ```

### POST /api/v1/auth/reset-password-request
- Request body:
  - email: string
- Response:
  - 200 OK (anche se email inesistente, per non rivelare dati)
  - 429 Too Many Requests (blocco tentativi reset)
- Funzione: genera token reset password, invia token via email
- Esempio:
  Request:
  ```json
  {
    "email": "utente@example.com"
  }
  ```

### POST /api/v1/auth/reset-password
- Request body:
  - token: string (token reset ricevuto via email)
  - newPassword: string (min 8 caratteri, lettere+numeri)
- Response:
  - 200 OK (password aggiornata)
  - 400 Bad Request (token scaduto o già usato, input non valido)
- Esempio:
  Request:
  ```json
  {
    "token": "reset-token-string",
    "newPassword": "NewPass123"
  }
  ```

### GET /api/v1/user/profile
- Protetto da JWT
- Response:
  - 200 OK con dati utente minimi (es. userId, email)
  - 401 Unauthorized (token mancante/scaduto)
- Esempio:
  Header: Authorization: Bearer jwt-token-string

## 6. Business logic
- Registrazione con validazione formale e unicità email, password hashata con bcrypt.
- Login verifica credenziali, blocco dopo 5 tentativi errati per 15 minuti.
- Generazione token JWT con userId e durata configurabile.
- Middleware verifica token JWT su chiamate protette.
- Reset password con token temporaneo inviato via email, unico e non riutilizzabile.
- Limitazione tentativi per login e reset password.
- Logging di eventi critici: login, reset, accessi protetti, errori.
- Monitoraggio tentativi sospetti per sicurezza aggiuntiva.
- Gestione ed escalation errori input, token e casi limite.
- Resilienza con retry/fallback su database temporaneamente non raggiungibile.

## 7. Persistenza e integrazioni
- PostgreSQL con tabella utenti dotata di colonne:
  - user_id (UUID PK)
  - email (varchar indice unico)
  - password_hash (varchar)
  - reset_token (varchar nullable)
  - reset_token_expiry (timestamp nullable)
  - tentativi_login (int)
  - blocco_login_fino_a (timestamp)
  - tentativi_reset (int)
  - blocco_reset_fino_a (timestamp)
- No integrazione esterna OAuth/SSO.
- Servizio SMTP per invio email reset password.
- Nessuna conferma email post-registrazione.
- Comunicazione unicamente su HTTPS.

## 8. Autenticazione e autorizzazione
- Autenticazione tramite JWT generati al login.
- Payload JWT minimale con campo userId.
- Durata token configurabile (default 30 min).
- Middleware per proteggere endpoint bloccando accesso senza token valido.
- Nessun refresh token o ruoli/permessi complessi gestiti.
- Blocchi temporanei su tentativi login e reset password basati su soglie definite.

## 9. Gestione errori
- Errori input con validazione e risposta 400 Bad Request.
- Errori credenziali su login con 401 Unauthorized.
- Blocchi dopo 5 tentativi con risposta 429 Too Many Requests.
- Gestione token JWT scaduti o invalidi con 401 Unauthorized.
- Token reset scaduto o già usato restituisce 400 con messaggio chiaro.
- Errori sistema e database con 500 Internal Server Error e logging.
- Messaggi errore non rivelano informazioni sensibili per sicurezza.

## 10. Strategia di test backend

### Authentication tests (tests/test_auth.py)
- test_register_user_success (unit): verifica registrazione con dati validi => 201
- test_register_user_invalid_email (unit): verifica validazione email => 400
- test_register_user_weak_password (unit): verifica validazione password => 400
- test_register_user_duplicate_email (integration): verifica unicità email => 400
- test_login_success (integration): login corretto genera token => 200 + token
- test_login_wrong_password_limit (integration): blocco dopo 5 tentativi => 429
- test_login_invalid_email (unit): email non esistente => 401
- test_jwt_token_expiry (unit): token invalido dopo scadenza => 401
- test_protected_endpoint_no_token (integration): accesso negato => 401

### Reset Password tests (tests/test_reset_password.py)
- test_reset_password_request_valid_email (integration): invio token => 200
- test_reset_password_request_too_many_attempts (integration): blocco => 429
- test_reset_password_success (integration): reset con token valido => 200
- test_reset_password_invalid_token (unit): token scaduto o usato => 400
- test_reset_password_weak_new_password (unit): validazione password => 400

### Performance tests (tests/test_performance.py)
- test_login_response_time (e2e): login sotto 500ms con dati realistici
- test_register_response_time (e2e): registrazione sotto 500ms

### Resilience tests (tests/test_resilience.py)
- test_database_unavailable_retry (integration): gestione perdita temporanea db

## 11. Rischi tecnici
- Definizione esatta di soglie tentativi e durata blocchi da consolidare.
- Affidabilità sistema email per invio token reset.
- Scalabilità e performance sotto picchi di carico da monitorare.
- Gestione perdita temporanea e degrado del database.
- Mancanza conferma email aumenta rischio account non verificati.
- Assenza di refresh token può limitare usabilità a token lunghi.
- Logging e monitoring devono bilanciare dettaglio e performance.

## 12. Struttura file proposta
```
authentication-backend/
  Main.java                         # Entry point — def main()
  config/
    Settings.java                  # Configurazione app — class Settings { tokenDuration, loginAttemptsLimit }
  models/
    User.java                     # Modello User — class User { uuid userId, String email, String passwordHash, ...}
    ResetToken.java               # Modello token reset password
  repositories/
    UserRepository.java           # Accesso DB utenti — interface UserRepository { findByEmail(), save(), update() }
    ResetTokenRepository.java     # Gestione token reset
  services/
    UserService.java              # Logica registrazione utente — registerUser(), validatePassword()
    AuthService.java              # Login e JWT — loginUser(), generateJWT(), validateJWT()
    ResetPasswordService.java     # Reset password — requestReset(), resetPassword()
    EmailService.java             # Invio email — sendResetEmail()
  middleware/
    JWTAuthMiddleware.java        # Validazione token JWT sui percorsi protetti
    RateLimitMiddleware.java      # Controllo e blocco tentativi login/reset
  controllers/
    AuthController.java           # Endpoint auth — register(), login(), resetPasswordRequest(), resetPassword()
    UserController.java           # Endpoint utente protetto — getProfile()
  logging/
    Logger.java                   # Gestione logging strutturato — logEvent(String eventType, Map details)
  monitoring/
    SecurityMonitor.java          # Monitoraggio tentativi sospetti e alert
  exceptions/
    CustomException.java          # Eccezioni personalizzate
    ExceptionHandler.java         # Gestione centralizzata errori REST
  tests/
    test_auth.java                # Test unit/integration login e registrazione
    test_reset_password.java      # Test reset password
    test_performance.java         # Test performance login e registrazione
    test_resilience.java          # Test resilienza DB temporaneo
```

## 13. Piano di implementazione
1. Progettazione schema DB e creazione tabelle PostgreSQL (User, ResetToken).
2. Implementazione modelli e repository per accesso dati.
3. Sviluppo modulo registrazione con validazione e hashing password.
4. Implementazione login con gestione errori, limitazioni e token JWT.
5. Realizzazione middleware JWT per protezione endpoint.
6. Sviluppo flusso reset password: richiesta token, invio email, reset con validazione.
7. Implementazione limitatori di tentativi e blocchi temporanei.
8. Aggiunta logging strutturato per eventi critici.
9. Costruzione monitoraggio sicurezza.
10. Centralizzazione gestione errori per risposte API chiare.
11. Test unitari e di integrazione per tutte le funzionalità.
12. Test di performance e ottimizzazione per mantenere risposte sotto 500 ms.
13. Implementazione resilienza e test per perdita temporanea del database.
14. Revisione finale, documentazione e rilascio MVP conforme specifiche.