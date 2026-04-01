MODULE: BACKEND VERSION: 1
## 1. Obiettivo backend
Progettare e implementare un backend API sicuro e scalabile per la gestione dell’autenticazione utenti, basato su FastAPI e PostgreSQL, che consenta registrazione, login con email e password, gestione di token JWT per autenticazione token-based, protezione degli endpoint via token, e un processo base di reset password tramite email. Il sistema deve integrare meccanismi fondamentali di sicurezza, logging degli eventi critici e garantire consistenza dati tramite transazioni, rispettando i vincoli tecnologici indicati e offrendo una base solida e riusabile per futuri ampliamenti.

## 2. Assunzioni tecniche
- Backend implementato con FastAPI (Python) come framework principale.
- Database relazionale PostgreSQL per persistenza dati.
- Gestione autenticazione tramite token JWT firmati (preferibilmente HS256).
- Comunicazione esclusivamente via HTTPS.
- Password utente salvate con hashing sicuro (bcrypt o argon2).
- Reset password tramite email esterna inviando token temporaneo (JWT scaduto o UUID sicuro).
- Nessuna gestione ruoli, multi-tenant, refresh token o revoca.
- Protezione basica contro attacchi comuni: brute force (rate limiting, blocco temporaneo), SQL injection, CSRF.
- Logging dedicato per eventi critici (login falliti, registrazioni, reset password).
- Uso di transazioni DB per atomicità nelle operazioni critiche (es. registrazione, reset password).
- Endpoint protetti accessibili solo tramite validazione token JWT valido e attivo.
- Non richiesto frontend o interfacce di autenticazione avanzate.
- Politiche di sicurezza configurabili per tentativi falliti e durata blocco account.

## 3. Architettura backend
Il sistema sarà organizzato come API RESTful in FastAPI, con architettura modulare basata su livelli separati: 
- livello di routing e gestione endpoint, 
- livello di service/business logic, 
- livello di accesso dati (repository/data access layer), 
- e componenti di sicurezza (JWT manager, password hashing, rate limiting middleware).  
L’architettura supporterà la scalabilità orizzontale con sessioni stateless basate su JWT e garantirà consistenza tramite transazioni PostgreSQL.
Un sistema di logging centralizzato traccerà eventi critici, mentre la configurazione di HTTPS sarà garantita a livello di deployment (proxy/reverse proxy o certificati SSL integrati).  
Error handling uniforme restituirà risposte generiche senza leak di informazioni sensibili.

## 4. Moduli e responsabilità
- **User Module**: gestione dati utente, schemi dati, CRUD relativi a registrazione e aggiornamento password.
- **Auth Module**: login, generazione JWT, validazione token, middleware di protezione endpoint.
- **Reset Password Module**: generazione token reset, invio email, validazione token, cambio password.
- **Security Module**: hashing password (bcrypt/argon2), protezioni brute force (rate limiting e blocco temporaneo account), validazioni input/sanitizzazione.
- **Database Layer**: interfaccia con PostgreSQL, gestione transazioni e query sicure.
- **Logging Module**: raccolta e memorizzazione eventi critici.
- **API Layer**: definizione e gestione endpoint REST con validazioni e sicurezza.
- **Config Module**: gestione configurazioni (JWT secret, scadenze token, parametri sicurezza, rate limiting, database).
  
## 5. API principali
- **POST /register**  
  Descrizione: registrazione nuovo utente con email e password.  
  Input: email (valida), password (rispetta criteri sicurezza).  
  Output: successo o errore generico.  

- **POST /login**  
  Descrizione: login e generazione token JWT.  
  Input: email, password.  
  Output: token JWT firmato con user id e scadenza.  

- **POST /password-reset/request**  
  Descrizione: richiesta reset password (invia token temporaneo via email).  
  Input: email registrata.  
  Output: messaggio generico di esito (non rivelare se email esiste o no).  

- **POST /password-reset/confirm**  
  Descrizione: validazione token reset + cambio password con hashing.  
  Input: token reset, nuova password.  
  Output: successo o errore generico.  

- **GET /protected-resource (esempio)**  
  Descrizione: esempio di endpoint protetto da token JWT valido.  
  Input: header Authorization Bearer token.  
  Output: dati o errore 401.  
  
## 6. Business logic
- Registrazione: verifica univocità email, validazione password, hashing sicuro, salvataggio utente in transazione.  
- Login: verifica credenziali, controllo blocco account, logging tentativi falliti, generazione JWT con claims minimi e scadenza configurabile.  
- Middleware verifica token JWT autenticità e integrità, validità scadenza e fornisce contesto utente agli endpoint.  
- Reset password: generazione token reset a breve scadenza, invio sicuro via email (layer esterno), salvataggio temporaneo token o gestione stateless con JWT. Validazione token reset prima del cambio password, hashing nuova password, logging evento.  
- Implementazione protezioni brute force con limitazione tentativi sia su login che reset password, blocco o delay incrementale configurabile.  
- Validazioni input tramite Pydantic + ulteriori sanitizzazioni per prevenzione injection e CSRF (anche se CSRF meno critica in API REST stateless).  

## 7. Persistenza e integrazioni
- PostgreSQL come DB relazionale, schema utenti con almeno campi: id (PK), email (unique), password_hash, timestamp creazione/aggiornamento, flag blocco account e contatori tentativi falliti.  
- Transazioni per operazioni critiche (registrazione, cambio password) per coerenza dati.  
- Integrazione con sistema email esterno per invio token reset (layer dedicato da configurare, es SMTP o servizio esterno).  
- Assicurare query parametrizzate per evitare SQL injection.  

## 8. Autenticazione e autorizzazione
- Autenticazione tramite JWT firmati, contenenti user id, scadenza configurabile (es. 1 ora).  
- Firma token con algoritmo HS256 usando segreto configurabile.  
- Middleware FastAPI per estrazione token da header Authorization, validazione firmata, scadenza e stato utente (es blocco).  
- Accesso negato con 401 per token assenti, invalidi o scaduti.  
- Nessuna autorizzazione granuale o gestione ruoli prevista.  
- Nessun refresh o revoca token implementati.  

## 9. Gestione errori
- Risposte uniformi e generiche sugli errori di autenticazione e input per prevenire informazioni sensibili.  
- Messaggi di errore standardizzati (es. "Credenziali non valide", "Token non valido o scaduto").  
- Gestione eccezioni interne con logging degli errori critici senza leakare stack trace o dettagli all’utente.  
- Codici HTTP appropriati (400 per input errati, 401 per errori autenticazione, 500 per errori server interni).  
- Sanitizzazione output per evitare leak di dati sensibili.  

## 10. Strategia di test backend
- Test unitari per logica business e hashing password.  
- Test di integrazione per endpoint REST (registrazione, login, reset password).  
- Test di sicurezza con casi edge:  
  - registrazioni con email duplicate,  
  - login con password errate multiple per blocco account,  
  - reset password con token scaduti o invalidi,  
  - tentativi multipli di reset simultanei,  
  - test rate limiting e bruteforce mitigation.  
- Test di validazione input e sanitizzazione dati.  
- Test transazionali per rollback in caso di errori durante operazioni critiche.  
- Test endpoint protetti per accesso con token valido e senza token.  

## 11. Rischi tecnici
- Ambiguità tecnologica da confermare: FastAPI vs Java.  
- Gestione e formato token reset password poco definita: scelta tra JWT temporaneo o UUID sicuro.  
- Politiche blocco account e rate limiting da definire con precisione.  
- Mancanza di refresh o revoca token limita flessibilità e sicurezza a lungo termine.  
- Potenziali problemi concurrency su reset password e registrazione multiple simultanee.  
- Assenza validazione obbligatoria email post-registrazione potrebbe portare a account con email non valide.  
- Necessità di configurazione HTTPS a livello deployment e compliance.  
- Performance target e SLA non specificati, da chiarire in seguito.  

## 12. Struttura file proposta
```
/app
  /api
    auth.py           # endpoint login, register
    password_reset.py # endpoint reset password
    protected.py      # esempi endpoint protetti
  /core
    config.py         # configurazioni globali
    security.py       # hashing password e JWT helper
    logger.py         # setup logging
  /db
    models.py         # definizione modelli SQLAlchemy
    repository.py     # accesso DB, transazioni
  /services
    auth_service.py   # business logic autenticazione
    user_service.py   # logica registrazione, gestione utenti
    reset_service.py  # logica password reset
  /middlewares
    jwt_middleware.py # validazione token JWT
    rate_limiter.py   # protezione brute force
  /schemas
    user.py           # Pydantic schemas per input/output
  /tests
    unit/             # test unitari
    integration/      # test integrazione API
main.py               # entrypoint FastAPI
requirements.txt      # dipendenze  
```

## 13. Piano di implementazione
1. Conferma definitiva stack: formalizzare scelta FastAPI vs Java.  
2. Definizione schema dati DB utente (email, password hash, contatori tentativi, timestamp).  
3. Setup progetto FastAPI con struttura base, configurazione DB e logging.  
4. Implementazione endpoint registrazione (validazione, hashing, salvataggio).  
5. Implementazione login (verifica, token JWT, logging errori).  
6. Middleware validazione JWT per endpoint protetti.  
7. Implementazione endpoint reset password: generazione token, invio email, validazione e cambio password.  
8. Aggiunta protezioni di sicurezza: rate limiting, blocco temporaneo account.  
9. Implementazione validazioni input/sanitizzazione dati globali.  
10. Configurazione HTTPS a livello deployment e gestione certificati.  
11. Test completi unitari e integrazione: criteri edge, concurrency e simili.  
12. Documentazione API ufficiale e criteri accettazione.  
13. Rilascio MVP backend conforme specifiche, pronto per integrazione frontend futura.