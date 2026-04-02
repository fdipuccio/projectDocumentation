MODULE: BACKEND VERSION: 1
## 1. Obiettivo backend
Realizzare un backend API RESTful per la gestione dell’autenticazione utenti che consenta: registrazione con email e password, login con successiva generazione di token JWT per sessioni stateless, protezione degli endpoint sensibili tramite validazione del token JWT, reset password tramite link unico inviato via email. Il sistema garantisce sicurezza robusta tramite hashing sicuro delle password, firma dei token JWT con secret key, mitigazione base di attacchi brute force, e log anonimi. Deve inoltre supportare scalabilità orizzontale e un monitoraggio minimo.

## 2. Assunzioni tecniche
- Stack tecnologico: Java (versione compatibile con librerie richieste), REST API, PostgreSQL DB, JWT per gestione token, Bear da approfondire per integrazione.
- Solo backend, senza interfacce frontend.
- La scadenza del token JWT è impostata a 1 ora come default.
- Reset password avviene tramite codice unico temporaneo via email; sistema email esterno presupposto.
- Non sono previsti ruoli o permessi, solo semplice controllo di autenticazione.
- Rate limiting da implementare in modo basilare per login e reset password, dettaglio da definire.
- Logging e monitoraggio limitati a informazioni anonime e relative a eventi critici.
- Scalabilità orizzontale garantita tramite architettura stateless e DB PostgreSQL centralizzato.

## 3. Architettura backend
Architettura modulare a componenti:
- Layer REST API per esposizione endpoint.
- Service layer contenente logica di business (es. validazione, generazione token).
- Repository layer per interazione con PostgreSQL tramite ORM o JDBC.
- Middleware o filtro per verifica token JWT sulle richieste agli endpoint protetti.
- Componente di integrazione per invio email reset password.
Il backend sarà containerizzabile per consentire scalabilità orizzontale.

## 4. Moduli e responsabilità
- Modulo User Management: gestione registrazione, autenticazione, hashing password, controllo unicità.
- Modulo Token Management: generazione, firma, validazione, scadenza token JWT.
- Modulo Reset Password: generazione codice temporaneo, invio email, verifica codice, cambio password.
- Modulo Security: mitigazione brute force, rate limiting base.
- Modulo Logging & Monitoring: registrazione accessi, fallimenti, anomalie di sicurezza.
- Modulo Configurazione: gestione parametri sistema, secret key JWT, configurazioni email.
- Integrazione Bear: da definire (probabile supporto o wrapper tecnico).

## 5. API principali

### POST /api/v1/auth/register
**Request Body**:  
```json
{
  "email": "string (email, required, unique, max 255)",
  "password": "string (required, min 8, max 128)"
}
```
**Response Body**:  
- 201 Created  
```json
{
  "message": "User registered successfully."
}
```
- 400 Bad Request (validation errors, email duplicate)

**Esempio Request**  
```json
{
  "email": "user@example.com",
  "password": "StrongPass123"
}
```
**Esempio Response**
```json
{
  "message": "User registered successfully."
}
```

---

### POST /api/v1/auth/login
**Request Body**:  
```json
{
  "email": "string (email, required)",
  "password": "string (required)"
}
```
**Response Body**:  
- 200 OK  
```json
{
  "token": "jwt_token_string",
  "expires_in": 3600
}
```
- 401 Unauthorized (invalid credentials)

**Esempio Request**
```json
{
  "email": "user@example.com",
  "password": "StrongPass123"
}
```
**Esempio Response**
```json
{
  "token": "eyJhbGciOiJIUzI1N...",
  "expires_in": 3600
}
```

---

### GET /api/v1/user/profile  (endpoint protetto es. info utente)
**Header**: Authorization: Bearer {token}  
**Response Body**:  
- 200 OK  
```json
{
  "email": "user@example.com",
  "registered_at": "2024-01-01T12:00:00Z"
}
```
- 401 Unauthorized (token mancante/scaduto/invalid)

---

### POST /api/v1/auth/reset-password/request
**Request Body**:  
```json
{
  "email": "string (email, required)"
}
```
**Response Body**:  
- 200 OK (indipendentemente esistenza email)  
```json
{
  "message": "If the email exists, a reset link has been sent."
}
```

---

### POST /api/v1/auth/reset-password/confirm
**Request Body**:  
```json
{
  "reset_code": "string (required)",
  "new_password": "string (required, min 8, max 128)"
}
```
**Response Body**:  
- 200 OK  
```json
{
  "message": "Password changed successfully."
}
```
- 400 Bad Request (reset_code invalid/scaduto)

## 6. Business logic
- Registrazione verifica validità email, formato password, unicità email, poi hash password e salva utente.
- Login verifica utente e password (bcrypt/Argon2 verify), se ok genera JWT con claims (email, iat, exp), firma con secret.
- Token JWT è controllato da filtro/middleware per accesso a endpoint protetti.
- Reset password genera codice univoco temporaneo (es UUID + expiry), memorizzato DB, invio email con link, conferma reset modifica password con hashing.
- Mitigazione brute force con contatore tentativi falliti per IP/utente e blocco temporaneo.
- Logging anonimizzato degli accessi, errori login, richieste reset.
- Monitoraggio trigger alert su errori critici o comportamenti anomali.

## 7. Persistenza e integrazioni
- PostgreSQL DB con schema utenti: id UUID PK, email unico, password_hash, reset_code, reset_expiry timestamp.
- ORM o JDBC template per accesso DB.
- Integrazione SMTP o servizio email esterno (configurabile) per invio reset password.
- Bear da approfondire e integrare a definizione tecnica.

## 8. Autenticazione e autorizzazione
- Autenticazione basata su email/password con JWT per sessioni stateless.
- Autorizzazione semplice: accesso endpoint protetti solo se JWT valido e non scaduto.
- Nessuna gestione ruoli/permessi.
- Protezione CSRF/XSS da parte client (non frontend) ma backend deve sanificare input e proteggere da injection.
- Secret key robusta per firma token JWT configurata esternamente.
- Politiche di rate limiting e blocchi temporanei per ridurre brute force.

## 9. Gestione errori
- Risposta standardizzata con codici HTTP corretti e messaggi chiari (es. 400 Bad Request con dettagli invalidazioni).
- Errori interni 500 con log senza esposizione di stack trace all’utente.
- Messaggi di errore non dettagliano se email esiste o meno (reset password).
- Logging eventi errore e sicurezza senza dati sensibili (mai password o token raw).
- Timeout e failover per componenti esterni (es. email).

## 10. Strategia di test backend

### Modulo Registrazione
- test_register_user_success (integration): verifica registrazione con dati validi -> 201 Created.
- test_register_user_duplicate_email (integration): verifica errore su email duplicata -> 400.
- test_register_user_invalid_email_format (unit): validazione formato email.
- test_register_user_password_strength (unit): password min length.

### Modulo Login
- test_login_success (integration): login corretto genera JWT.
- test_login_invalid_credentials (integration): errore login con credenziali errate.
- test_login_bruteforce_block (integration): verifica blocco dopo tentativi multipli falliti.

### Modulo JWT
- test_token_generation_correct_claims (unit): token contiene claims attesi.
- test_token_validation_success (integration): filtro consente accesso con JWT valido.
- test_token_validation_expired (integration): accesso negato con token scaduto.

### Modulo Reset Password
- test_reset_request_email_sent (integration): invio email con codice reset.
- test_reset_confirm_success (integration): conferma reset con codice valido.
- test_reset_confirm_invalid_code (integration): errore reset con codice non valido o scaduto.

### Modulo Logging e Monitoring
- test_logging_events_recorded (integration): verificare che eventi accesso/fallimenti siano loggati.
- test_monitoring_alert_triggers (integration): simulare errori e verificare alert.

## 11. Rischi tecnici
- Mancata definizione precisa di utilizzo e integrazione Bear, possibile ritardo.
- Parametri di rate limiting non definiti; implementazione troppo rigida o troppo blanda.
- Dipendenza da sistema esterno di invio email non dettagliato può impattare reset password.
- Scalabilità reale da validare in ambiente cluster; problemi di concorrenza DB e caching token.
- Eventuale futura estensione per ruoli/perms può richiedere refactoring architetturale.
- Gestione sicura delle chiavi segrete JWT e loro rotazione non esplicitata nella spec.

## 12. Struttura file proposta
```plaintext
auth-backend/
  Application.java                # Entry point - def main()
  config/
    Settings.java                # Configurazioni sistema - class Settings
    JwtConfig.java               # Configurazione JWT (secret, expiry, etc.)
  models/
    User.java                   # Entity utente DB - class User
    ResetPasswordToken.java      # Entity reset password - class ResetPasswordToken
  repository/
    UserRepository.java          # Accesso dati utenti - interface UserRepository
    ResetPasswordRepository.java # Accesso dati reset token - interface ResetPasswordRepository
  service/
    UserService.java             # Logica registrazione e login - class UserService
    JwtService.java              # Generazione e validazione JWT - class JwtService
    ResetPasswordService.java    # Gestione reset password - class ResetPasswordService
    SecurityService.java         # Mitigazione brute force e rate limiting - class SecurityService
    EmailService.java            # Invio email reset - class EmailService
  controller/
    AuthController.java          # Endpoint registrazione, login, reset password - class AuthController
    UserController.java          # Endpoint protetti es. profilo utente - class UserController
  filter/
    JwtAuthenticationFilter.java # Filtro autenticazione JWT - class JwtAuthenticationFilter
  util/
    PasswordHasher.java          # Hashing password sicuro - class PasswordHasher
    Validator.java               # Validazioni input - class Validator
  logging/
    AuditLogger.java             # Logging accessi e errori - class AuditLogger
  monitoring/
    MetricsService.java          # Raccolta metriche e alert - class MetricsService
  tests/
    AuthControllerTest.java      # Test integrazione endpoint auth
    UserServiceTest.java         # Unit e integration test UserService
    JwtServiceTest.java          # Unit test JWT
    ResetPasswordServiceTest.java# Test reset password
    SecurityServiceTest.java     # Test mitigazioni sicurezza
```

## 13. Piano di implementazione
1. Approfondire e definire integrazione con Bear (task critico da chiarire prima di sviluppo).
2. Progettazione schema DB utenti e reset token.
3. Implementazione API registrazione utenti con validazioni e hashing password.
4. Creazione endpoint login con generazione token JWT firmato.
5. Sviluppo filtro middleware per validazione JWT su endpoint sicuri.
6. Realizzazione flusso reset password: richiesta codice via email, generazione codice temporaneo, conferma reset con cambio password.
7. Implementazione sicurezza password con bcrypt o Argon2 configurabili.
8. Introduzione meccanismo base di rate limiting e mitigazione brute force.
9. Setup logging anonimo e sicuro per eventi sensibili di accesso e reset.
10. Aggiunta monitoraggio base e alert su errori di sicurezza critici.
11. Test funzionali e di sicurezza su tutti i moduli (token, concorrenza, edge cases).
12. Stesura completa della documentazione backend e API per rilascio.