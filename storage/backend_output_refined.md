MODULE: BACKEND VERSION: 1
## 1. Obiettivo backend
Sviluppare un backend API RESTful sicuro, scalabile e manutenibile per la gestione dell’autenticazione utenti tramite FastAPI e PostgreSQL. Il sistema deve permettere la registrazione degli utenti, login con email e password, generazione e validazione di token JWT per autenticazione token-based, protezione degli endpoint tramite validazione token, e un processo base di reset password con invio token temporaneo via email. Il backend includerà meccanismi di sicurezza quali hashing password (bcrypt/argon2), mitigazione brute force con rate limiting e blocco temporaneo account, sanitizzazione input, logging eventi critici e gestione transazioni DB per garantire consistenza operazioni.

## 2. Assunzioni tecniche
- Backend implementato definitivamente con FastAPI (Python).  
- Persistenza dati su PostgreSQL, accesso tramite ORM compatibile (es. SQLAlchemy o equivalent), utilizzo transazioni DB per operazioni critiche.  
- Uso di JWT (HS256) per token di sessione e di reset password; formato e gestione token di reset password da formalizzare ma previsto JWT stateless per semplicità.  
- Password hashed usando bcrypt o argon2 (configurabile).  
- Limitazione tentativi falliti di login e reset password con soglie configurabili (rate limiting e blocco temporaneo).  
- Comunicazione HTTPS obbligatoria gestita a livello deployment (certificate SSL gestiti esternamente).  
- Validazione e sanitizzazione input con Pydantic, uso di query parametrizzate per difesa da SQL injection, architettura stateless per mitigazione CSRF.  
- Logging eventi critici secondo linee guida aziendali, inclusa definizione futura di retention e privacy compliance.  
- Gestione errori consistente con risposte generiche per non esporre dettagli sensibili.  
- Mancata gestione di ruoli, refresh token, revoca token, o validazione email post-registrazione (scope MVP).  

## 3. Architettura backend
Architettura modulare a livelli con separazione chiara tra:  
- Layer API (endpoint FastAPI con validazione input/output Pydantic)  
- Business logic (gestione autenticazione, registrazione, reset password, validazioni sicurezza)  
- Persistenza (ORM per accesso a PostgreSQL, gestione transazioni)  
- Sicurezza (hashing password, gestione JWT, middleware validazione token, rate limiting)  
- Logging (servizio centrale per eventi critici)  
- Integrazione email esterna per invio token reset password  
Il sistema è progettato per essere scalabile orizzontalmente tramite replica dei servizi FastAPI e database PostgreSQL con connessioni gestite.  

## 4. Moduli e responsabilità
- **User Registration Module**: Validazione email/password, hashing password, salvataggio utente con transazione DB.  
- **Authentication Module**: Verifica credenziali, generazione token JWT, logging accessi e fallimenti login.  
- **Authorization Middleware**: Validazione accesso token JWT per endpoint protetti con gestione errori 401.  
- **Reset Password Module**: Generazione token temporaneo JWT per reset, invio email con token, verifica token per cambio password con hashing.  
- **Security Module**: Implementazione rate limiting, blocco account temporaneo dopo tentativi falliti, sanitizzazione input, difesa da SQL injection e CSRF.  
- **Logging Module**: Registrazione eventi critici (login fallito, reset richiesti, registrazione), gestione formati logging e privacy compliance.  
- **Error Handling Module**: Gestione uniforme delle eccezioni, elaborazione risposte generiche senza leak informativi.  

## 5. API principali
- **POST /register**: Registrazione nuovo utente con body contenente email e password (validazione, verifica duplicati, hashing).  
- **POST /login**: Login utente con email e password, ritorno token JWT firmato (claim user id, scadenza configurabile).  
- **POST /reset-password/request**: Richiesta reset password con email registrata, generazione token temporaneo e invio via email.  
- **POST /reset-password/confirm**: Cambio password con token reset e nuova password, validazione token e hashing password.  
- **GET /protected-resource** (esempio): Endpoint protetto da autenticazione JWT, accessibile solo con token valido (middleware validazione).  

## 6. Business logic
- Validazione sintattica e semantica di email e password in registrazione.  
- Verifica unica email durante registrazione con lock ottimistico o serializzazione per gestire concorrenza.  
- Hashing password con bcrypt/argon2 su valori sicuri a lungo termine.  
- Verifica credenziali al login e generazione JWT contenente user id e claim exp configurabile.  
- Gestione tentativi falliti con incremento contatore e blocco temporaneo account configurabile dopo soglia.  
- Generazione token reset password JWT a breve scadenza (es. 15 min/1 ora), firmato e contenente user id.  
- Invio email esterno asincrono con token reset password.  
- Cambio password solo se token reset valido e non scaduto, con aggiornamento password hashed in transazione.  
- Registrazione eventi critici con dettaglio minimo ma sufficiente per audit e monitoraggio.  

## 7. Persistenza e integrazioni
- Database PostgreSQL come unico store persistente con modello tabella utente contenente campi minimi: id (UUID), email (unique), password_hash, timestamp creazione e aggiornamento, contatori tentativi login falliti e timestamp blocco account.  
- Uso ORM (ad es. SQLAlchemy) per accesso dati e supporto transazioni.  
- Gestione transazioni robuste per garantire rollback automatici su errori in registrazione e reset password.  
- Integrazione con servizio SMTP esterno per invio email reset password, con logging e gestione errori invio.  
- Configurazioni sensibili (chiave segreta JWT, parametri rate limiting) gestite tramite variabili ambiente e sistemi di secret management.  

## 8. Autenticazione e autorizzazione
- Autenticazione stateless via JWT HS256, con token firmato contenente user id e tempo scadenza configurabile (es. 1h, configurabile).  
- Middleware FastAPI per validazione token su header Authorization, parsing e verifica firma e scadenza.  
- Risposta 401 su token mancante, scaduto o non valido con messaggi generici senza leak informazioni.  
- Nessuna gestione livelli di autorizzazione o ruoli, scope limitato ad accesso autenticato vs non autenticato.  
- Nessun meccanismo refresh o revoca token implementato (out of scope).  
- Protezione base da CSRF tramite design API stateless e uso esclusivo di HTTPS.  
- Gestione blocco o rate limiting tentativi login e reset password per mitigare brute force.  

## 9. Gestione errori
- Risposte uniformi in formato JSON per errori, con campi standard (es. {"detail": "...messaggio generico..."}).  
- Nessuna esposizione di dettagli tecnici o stack trace nelle risposte API.  
- Log di errori server con tracciamento stack per analisi interna.  
- Gestione errori email esterna: logging errori invio, retry asincrono pianificato o segnalazione operativa (da definire successivamente).  
- ValidationError Pydantic gestite con messaggi sintetici.  
- Gestione errori DB con rollback automatico delle transazioni.  

## 10. Strategia di test backend
- Test unitari coprenti hashing password, generazione e validazione token JWT (sessione e reset).  
- Test di integrazione per endpoint principali (register, login, reset request/confirm, endpoint protetti).  
- Test di sicurezza su mitigazione brute force e rate limiting (tentativi multipli successivi).  
- Test concurrency su registrazione e reset password simultanei per verificare consistenza dati e rollback.  
- Test per evitare SQL injection e sanitizzazione input con payload malevoli.  
- Test CSRF via simulazione chiamate non autorizzate (anche se limitato da design API REST).  
- Test accesso endpoint protetti con token validi, invalidi, scaduti e assenti (verification risposta 401).  
- Test logging eventi critici per correttezza e assenza di leak informazioni.  
- Test end-to-end flusso reset password inclusi invio email simulato.  
- Include proposte future di test stress e carico per validare performance e scalabilità.  

## 11. Rischi tecnici
- Persistono rischi per gestione ambigua del token reset password (JWT vs UUID), necessita definizione operativa precisa.  
- Politiche di blocco account e rate limiting non completamente dettagliate, rischio buchi nella mitigazione bruteforce.  
- Assenza di gestione refresh/revoca token limita flessibilità e sicurezza a lungo termine.  
- Mancanza di validazione email post registrazione può favorire account con email inesistenti.  
- Dipendenza da livello deployment per HTTPS e gestione certificati senza dettagli operativi specifici.  
- Mancanza di piano dettagliato per rotazione chiavi segrete JWT e protezione configurazioni sensibili.  
- Gestione concorrente e race condition in registrazione e reset password resta un rischio da mitigare con test specifici.  
- Possibili problemi di affidabilità e gestione errori con sistema email esterno.  

## 12. Struttura file proposta
```
/app
  /api
    __init__.py
    routes_auth.py            # endpoint register/login/reset
    routes_protected.py       # endpoint protetti
  /core
    config.py                 # configurazioni, variabili ambiente
    security.py               # hashing, JWT generation/validation
    exceptions.py             # gestione errori personalizzati
  /db
    base.py                   # sessione DB, connessione, init ORM
    models.py                 # modello User
    crud.py                   # funzioni accesso dati
  /services
    auth_service.py           # logica autenticazione e registrazione
    reset_password_service.py # logica reset password
    email_service.py          # invio email con token reset
    rate_limiter.py           # gestione limit rate/block account
    logging_service.py        # gestione log eventi critici
  /middlewares
    auth_middleware.py        # middleware validazione JWT
    rate_limit_middleware.py  # middleware rate limiting
/tests
  test_auth.py                # test unit e integrazione autenticazione
  test_reset_password.py      # test reset password
  test_security.py            # test sicurezza (bruteforce, injection)
/main.py                      # avvio FastAPI app
```

## 13. Piano di implementazione
1. Formalizzare definitivamente la scelta FastAPI vs Java (previsto FastAPI).  
2. Definire schema token reset password: adozione JWT con breve scadenza, contenente user id, firmato con chiave segreta.  
3. Implementare modello utente e accesso dati con ORM e supporto transazioni.  
4. Costruire endpoint di registrazione /register con validazione, hashing e gestione duplicati.  
5. Implementare endpoint login /login con verifica e generazione token JWT a scadenza configurabile.  
6. Sviluppare middleware per validazione token JWT su endpoint protetti.  
7. Progettare endpoint /reset-password/request e /reset-password/confirm per processo reset password, integrazione email asincrona.  
8. Introdurre meccanismi di rate limiting e blocco account temporaneo configurabili.  
9. Integrare sistema logging per eventi critici (registrazione, login falliti, reset password).  
10. Applicare validazioni input avanzate con Pydantic e sanitizzazione, protezione base CSRF e SQL injection.  
11. Testare la concorrenza su registrazioni e reset simultanei tramite test automatici.  
12. Completare copertura test automatizzati e predisporre repository per test di carico in futuro.  
13. Documentare API dettagliate e criteri di accettazione, predisporre documentazione deployment HTTPS e sicurezza chiavi.  
14. Preparare pipeline monitoraggio e retention logging per compliance.  
15. Rilascio MVP con monitoraggio feedback operativo e pianificazione evolutiva (inclusione refresh token, validazione email post-registrazione, gestione chiavi segrete avanzata).