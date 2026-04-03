MODULE: BACKEND VERSION: 1

## 1. Obiettivo backend
Sviluppare un backend API REST in Java utilizzando il framework Bear per la gestione dell’autenticazione utenti. Il sistema permetterà registrazione, login con email e password, generazione e validazione di token JWT per sessioni stateless, protezione degli endpoint tramite JWT, reset password base con invio di link temporaneo via email. Il backend garantirà sicurezza con password hashate e salate, comunicazioni via HTTPS, validazione input per prevenzione injection e logging centralizzato conforme GDPR degli eventi di autenticazione.

## 2. Assunzioni tecniche
- Backend esclusivamente Java su framework Bear con architettura REST API.
- Database PostgreSQL con schema utenti e gestione token reset password.
- Password memorizzate con algoritmo hash + salt (bcrypt o simile).
- JWT firmati con chiave segreta configurabile e scadenza fissa (1 ora).
- Comunicazioni protette da HTTPS a livello infrastrutturale.
- Middleware Bear per verifica token JWT applicato a tutti gli endpoint protetti.
- Endpoint pubblici solo per registrazione, login e reset password.
- Logging centralizzato con anonimizzazione dati personali e compliance GDPR.
- Nessun supporto per social login, ruoli, conferma email o reset avanzato.
- Gestione errori con messaggi generici per ragioni di sicurezza.
- Architettura stateless e scalabile orizzontalmente attraverso uso esclusivo di JWT per sessioni.

## 3. Architettura backend
Il backend si basa su un’architettura REST stateless:
- Controller REST espongono gli endpoint per autenticazione.
- Service layer gestisce logica business, hashing password, generazione JWT, validazioni, invio email.
- Repository per accesso a PostgreSQL con JPA o framework Bear ORM.
- Middleware Bear gestisce autenticazione JWT e protezione endpoints.
- Componente di logging centralizzato per eventi di autenticazione.
- Configurazioni esterne per chiavi JWT, SMTP, scadenze token, HTTPS.
- Flussi asincroni per invio email reset password con retry/fallback.
- Schema DB con tabelle utenti e token reset password relazionati tramite foreign key.
- Schema modulare e facile da estendere in futuro con ulteriori funzioni di sicurezza.

## 4. Moduli e responsabilità
- mod.user: gestione utenti, registrazione, hashing password, validazione input.
- mod.auth: login, generazione e verifica token JWT.
- mod.resetpassword: gestione token reset, invio email, sicurezza reset base.
- mod.middleware: filtro Bear per validazione JWT su percorsi protetti.
- mod.logging: gestione logging centralizzato anonimizzato e conforme GDPR.
- mod.config: configurazioni sicure per JWT, SMTP, HTTPS.
- mod.exception: gestione centralizzata errori con messaggi generici.
- mod.validation: componenti per validazione input per evitare injection.

## 5. API principali
- POST /api/v1/auth/register
  - Request: 
    ```json
    {
      "email": "string (RFC valid)",
      "password": "string (min 8 chars)"
    }
    ```
  - Response:
    - 201 Created
    - 400 Bad Request (validazione fallita)
    - 409 Conflict (email duplicata)
    ```json
    {
      "message": "User registered successfully"
    }
    ```
  - Esempio request:
    ```json
    {
      "email": "utente@example.com",
      "password": "P@s$w0rd123"
    }
    ```
  - Esempio response:
    ```json
    {
      "message": "User registered successfully"
    }
    ```

- POST /api/v1/auth/login
  - Request: 
    ```json
    {
      "email": "string (RFC valid)",
      "password": "string"
    }
    ```
  - Response:
    - 200 OK 
    - 400 Bad Request (input non valido)
    - 401 Unauthorized (credenziali errate)
    ```json
    {
      "token": "jwt_token_string",
      "expires_in": 3600
    }
    ```
  - Esempio request:
    ```json
    {
      "email": "utente@example.com",
      "password": "P@s$w0rd123"
    }
    ```
  - Esempio response:
    ```json
    {
      "token": "eyJhbGciOiJI...",
      "expires_in": 3600
    }
    ```

- POST /api/v1/auth/reset-password
  - Request:
    ```json
    {
      "email": "string (RFC valid)"
    }
    ```
  - Response:
    - 200 OK (messaggio generico indipendentemente dall’esito)
    - 400 Bad Request (input non valido)
    ```json
    {
      "message": "If the email exists, a reset link has been sent"
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
      "message": "If the email exists, a reset link has been sent"
    }
    ```

- GET /api/v1/protected-resource (esempio endpoint protetto)
  - Richiede header Authorization: Bearer <token>
  - Response:
    - 200 OK se token valido
    - 401 Unauthorized se token mancante o invalido
    ```json
    {
      "data": "Protected data"
    }
    ```

## 6. Business logic
- Registrazione: verifica unicità email con lock DB, hash+salt password, salvataggio record utente.
- Login: verifica presenza utente, confronto password hash, generazione token JWT con contenuto minimale e scadenza.
- JWT: chiave segreta sicura, scadenza fissa 1 ora, nessun meccanismo di revoca attivo.
- Middleware valida token JWT per accesso endpoint protetti, rifiuta senza token o token scaduto.
- Reset password: genera token univoco temporaneo e lo memorizza con scadenza, invia link tramite email, risponde con messaggio anonimo per non rivelare esistenza email.
- Logging centralizzato registra eventi login, reset e errori, mascherando dati sensibili secondo GDPR.
- Validazione input in ingresso per prevenzione injection con filtri regex e sanitizzazione.

## 7. Persistenza e integrazioni
- PostgreSQL con tabella utenti:
  - id UUID PK
  - email VARCHAR unica e indicizzata
  - password_hash TEXT
  - timestamps creazione e aggiornamento
- Tabella reset_password_tokens:
  - token UUID PK
  - user_id FK verso utenti
  - expiration timestamp
  - indicizzazione su expiration
- Integrazione SMTP per invio email reset password con retry e fallback gestiti da componente apposito.
- Configurazione chiavi JWT e parametri via file di configurazione esterni sicuri.
- Logging centralizzato verso sistema esterno compatibile GDPR.

## 8. Autenticazione e autorizzazione
- Autenticazione tramite JWT firmato con segreto configurato, scadenza 1 ora.
- Middleware Bear intercetta richieste endpoint protetti, verifica presenza e validità token JWT.
- Endpoint pubblici: registrazione, login, reset-password senza autenticazione.
- Nessuna gestione ruoli o permessi differenziati, accesso solo autenticato o pubblico.
- Nessun social login o federated authentication.

## 9. Gestione errori
- Risposte con codici HTTP appropriati:
  - 201 Created per registrazioni ok.
  - 200 OK per login e reset password ok.
  - 400 Bad Request per dati input invalidi.
  - 401 Unauthorized per token JWT mancanti o invalidi.
  - 409 Conflict per email duplicata in registrazione.
  - 500 Internal Server Error per errori interni.
- Messaggi di errore generici per reset password e login per non rivelare dettagli di sicurezza.
- Logging degli errori senza esposizione di dati sensibili.

## 10. Strategia di test backend
- test_register_user_success (unit/integration)
  - Verifica registrazione con email valida e password solida.
  - Input: email valida, password 8+ caratteri.
  - Expected: 201 Created, utente nel DB con password hashata.
- test_register_user_duplicate_email (unit/integration)
  - Verifica errore su email duplicata.
  - Input: email già esistente.
  - Expected: 409 Conflict.
- test_login_success (unit/integration)
  - Verifica login con corrette credenziali.
  - Input: email esistente, password corretta.
  - Expected: 200 OK, token JWT valido.
- test_login_invalid_password (unit/integration)
  - Verifica errore login con password errata.
  - Input: email esistente, password errata.
  - Expected: 401 Unauthorized.
- test_reset_password_email_sent (integration)
  - Verifica invio email reset per email esistente.
  - Input: email esistente.
  - Expected: 200 OK, email inviata.
- test_reset_password_generic_response (integration)
  - Verifica risposta generica anche per email non registrata.
  - Input: email inesistente.
  - Expected: 200 OK.
- test_protected_endpoint_token_validation (integration)
  - Verifica accesso endpoint protetto solo con token JWT valido.
  - Input: richieste con/ senza token valido.
  - Expected: 200 OK o 401 Unauthorized.
- test_input_validation_injection_prevention (unit)
  - Test stringhe malformate o SQL injection nei campi email/password.
  - Expected: 400 Bad Request.
- test_logging_anonymization (unit)
  - Validazione che logging mascheri dati sensibili.
- test_jwt_expiration (unit)
  - Verifica scadenza token a 1 ora.

## 11. Rischi tecnici
- Alta: Mancanza meccanismi anti-brute force espone login a tentativi di forza bruta.
- Alta: Assenza revoca token JWT limita possibilità logout remoto e invalidamento token compromessi.
- Media: Race condition sporadiche in registrazione non totalmente mitigata oltre lock DB.
- Media: Possibili problemi affidabilità invio email reset in mancanza di fallback robusto.
- Media: Sincronizzazione oraria cluster potrebbe impattare validità token JWT.
- Media: Registrazioni con email errate per assenza meccanismo conferma email.
- Alta: Compliance GDPR logging richiede trattamento attento per evitare esposizione dati sensibili.

## 12. Struttura file proposta
```  
project_root/
  src/
    main/
      java/
        com/
          company/
            backend/
              Application.java              # Entry point — public static void main(String[] args)
              config/
                JwtConfig.java               # Configurazione JWT — class JwtConfig
                SmtpConfig.java              # Configurazione SMTP – class SmtpConfig
              controllers/
                AuthController.java          # Endpoint autenticazione — register(), login(), resetPassword()
                ProtectedController.java     # Endpoint protetti esempio — getProtectedResource()
              models/
                User.java                   # Modello User JPA — class User {UUID id, String email, String passwordHash,...}
                ResetPasswordToken.java      # Modello Token reset — class ResetPasswordToken
              repositories/
                UserRepository.java         # Interfaccia JPA — findByEmail(), save()
                ResetPasswordTokenRepository.java # Gestione token reset
              services/
                UserService.java            # Logica utenti — registerUser(), hashPassword()
                AuthService.java            # Login e JWT — authenticate(), generateJwtToken()
                ResetPasswordService.java   # Reset password token, invio email
                EmailService.java           # Invio email SMTP con retry
              middleware/
                JwtAuthFilter.java          # Middleware verifica JWT — doFilter()
              logging/
                AuthLogger.java             # Logging eventi autenticazione anonimi
              exceptions/
                ApiExceptionHandler.java    # Gestione errori globale con risposte JSON generiche
              validation/
                InputValidator.java         # Validazione campi input email/password
  tests/
    auth/
      AuthControllerTest.java           # Test integrazione auth — test_register_user_success(), test_login_invalid_password()
    service/
      UserServiceTest.java              # Test unitari hashing password, validazioni input
      ResetPasswordServiceTest.java
    middleware/
      JwtAuthFilterTest.java            # Test middleware JWT validazione token
    logging/
      AuthLoggerTest.java               # Test logging anonimizzato compliance GDPR
```

## 13. Piano di implementazione
1. Definire lo schema database PostgreSQL con tabelle utenti e token reset con vincoli e indici.
2. Implementare servizio hashing/salting password (bcrypt).
3. Realizzare endpoint POST /register con validazioni e gestione errori.
4. Implementare endpoint POST /login con verifica credenziali, generazione token JWT firmato e scadenza.
5. Sviluppare middleware Bear JwtAuthFilter per protezione endpoint con verifica token JWT.
6. Implementare endpoint POST /reset-password per generare token reset, memorizzarlo e inviare link via email con retry/fallback.
7. Predisporre logging centralizzato anonimizzato per eventi autenticazione.
8. Configurare proprietà esterne (JWT, SMTP, HTTPS).
9. Implementare validazioni input per prevenzione injection.
10. Scrivere test unitari e di integrazione coprendo flussi positivi e scenari errore.
11. Collaborare con team infrastruttura per abilitare HTTPS e configurare ambiente.
12. Revisionare implementazione per compliance GDPR logging e sicurezza.
13. Documentare API e flussi per utilizzo frontend o futuri integrazioni.
14. Monitorare e pianificare implementazione futura meccanismi anti-brute force e revoca token JWT come successive migliorie.