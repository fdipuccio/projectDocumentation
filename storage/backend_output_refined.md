MODULE: BACKEND VERSION: 1
## 1. Obiettivo backend
Sviluppare un backend API REST utilizzando FastAPI (Python) per gestire un sistema di autenticazione utenti con funzionalità essenziali: registrazione con email e password, login che restituisce token JWT, protezione di endpoint riservati tramite autenticazione JWT, e implementazione di un flusso base di reset password tramite token. La soluzione deve garantire sicurezza, scalabilità enterprise-ready, e rispetto dei vincoli tecnologici (FastAPI, PostgreSQL, JWT).

## 2. Assunzioni tecniche
- Il backend sarà sviluppato esclusivamente con FastAPI e Python, privilegiando async dove possibile.
- PostgreSQL sarà il database relazionale per la persistenza dati utenti e token reset.
- Password hashing sicuro tramite librerie consolidate (es. bcrypt o Argon2).
- I token JWT useranno algoritmo HS256 con segreto configurabile e durata expiration esplicita.
- Reset password si basa su token univoci salvati in DB, con scadenza e meccanismo base di invalidazione.
- Invio email per reset password è out of scope.
- Validazione email e password minima; policy di complessità estesa può essere integrata successivamente.
- Il team è competente in Python/FastAPI dopo eventuale formazione, nonostante lo stack iniziale menzioni Java.
- Tutti i segreti (JWT secret, DB credentials) saranno gestiti in modo sicuro (variabili ambiente o vault).
- Documentazione API automatiche tramite OpenAPI saranno esposte.
- Logging standard di errore e audit logging base per azioni critiche.
- Non si gestiscono ruoli/autorizzazioni complesse oltre autenticazione base.

## 3. Architettura backend
Architettura modulare e stratificata:
- Layer Model: definizione ORM (SQLAlchemy/o async variant) per utenti e token reset.
- Layer Schema: validazione dati in input/output con Pydantic.
- Layer Service: logica business astratta per autenticazione, gestione utente, reset password e token.
- Layer Auth: gestione JWT, dependency e middleware per protezione endpoint.
- Layer Route: esposizione endpoint REST organizzati per funzionalità (/auth/..., /protected).
- Configurazione centralizzata per segreti e parametri (JWT secret, durata, DB url).
- Logging e gestione errori integrati a livello globale.
Questa architettura consente separazione delle responsabilità e facilita test e manutenzione.

## 4. Moduli e responsabilità
- models.py: modelli dati ORM per User e ResetToken con campi email, password_hash, token reset, timestamp creazione/modifica.
- schemas.py: Pydantic schemas per richiesta/risposta (UserCreate, UserLogin, TokenResponse, ResetRequest, ResetConfirm).
- auth.py: gestione JWT (creazione, decodifica, validazione), hashing password, verifica credenziali, dependency protettiva FastAPI.
- services.py: business logic di registrazione, login, generazione e verifica token reset, cambio password.
- routes/auth.py: endpoint REST /register, /login, /reset-password/request, /reset-password/confirm.
- routes/protected.py: endpoint esempio /protected protetto tramite JWT.
- config.py: gestione configurazioni centralizzate e variabili ambiente.
- database.py: setup connessione e sessione PostgreSQL async.
- main.py: applicazione FastAPI e montaggio router.
- utils.py: funzioni di supporto (es. generatore token reset, helper validazioni).
- logger.py: configurazione logging e audit logging minimale.

## 5. API principali
- POST /auth/register  
  Input: email, password  
  Output: conferma registrazione o errore  
  Funzionalità: crea utente con password hashed, controlla unicità email

- POST /auth/login  
  Input: email, password  
  Output: JWT token con claims user_id e expiration  
  Funzionalità: verifica credenziali, genera token JWT firmato

- POST /auth/reset-password/request  
  Input: email  
  Output: conferma creazione token reset (senza invio email)  
  Funzionalità: genera token reset univoco, lo salva in DB con timestamp

- POST /auth/reset-password/confirm  
  Input: reset token, nuova password  
  Output: conferma cambio password o errore (token non valido o scaduto)  
  Funzionalità: verifica token, aggiorna password hashed, invalida token

- GET /protected  
  Output: contenuto accessibile solo con JWT valido  
  Funzionalità: test protezione endpoint con autenticazione JWT

## 6. Business logic
- Registrazione: validazione email, controllo duplicati, hashing password con salt, persistenza utente.
- Login: verifica email esistente, confronto password hashed, creazione token JWT con expiraton.
- JWT: firma con HS256, inclusion claim standard (sub=user_id, exp), validazione su ogni request protetta.
- Reset password: generazione token random sicuro (UUID o simile), associazione token con timestamp creazione, controllo scadenza token (es. 1 ora), invalidazione dopo uso.
- Endpoint protetti tramite dependency che decodifica, valida token e carica user_id.
- Gestione errori e risposte chiare senza esporre dati sensibili.

## 7. Persistenza e integrazioni
- PostgreSQL via async ORM SQLAlchemy con sessione asincrona.
- Tabelle utenti con colonne: id, email(unique), password_hash, created_at, updated_at.
- Tabella token reset con token, user_id FK, created_at, used_flag o expired_flag.
- Connessione DB sicura con parametri da variabili ambiente.
- Nessuna integrazione esterna prevista (es. SMTP) per mailing.
- Repository pattern opzionale per isolamento DB da logica business.

## 8. Autenticazione e autorizzazione
- Autenticazione basata su JWT protetto da secret HS256.
- Token JWT include claims: sub (user_id), iat, exp (configurabile, es. 15-30 minuti).
- Middleware/Dependency FastAPI verifica presenza e validità token in header Authorization Bearer.
- Accesso consentito solo a endpoint protetti con token valido; 401 se mancante o invalido.
- Reset password usa token operativi salvati in DB con controllo scadenza e one-time use.
- Password hashing sicuro (bcrypt/Argon2) con salt individuali.
- Nessuna gestione ruoli o autorizzazioni avanzate, solo autenticazione base.

## 9. Gestione errori
- Risposte HTTP standard: 400 (bad request), 401 (non autorizzato), 404 (non trovato), 409 (conflitto), 500 (server error).
- Messaggi errori user-friendly, non rivelano dettagli sensibili o stacktrace.
- Validazione input con Pydantic con messaggi di errore chiari.
- Gestione eccezioni globale FastAPI per intercettare errori inattesi.
- Log errori con stacktrace limitato a log file, evitando esposizione su API.
- Audit logging per tentativi login, reset password e cambi di credenziali.
- Rate limiting non implementato nel MVP ma previsto in evoluzione.

## 10. Strategia di test backend
- Unit test su funzioni hashing, verifiche password, generazione e verifica token JWT.
- Unit test per business logic di registrazione, login, reset password token management.
- Test d’integrazione sugli endpoint REST per flussi corretti e casi negativi (email duplicata, token scaduto).
- Test negativi per input invalidi, password errate, token JWT falsificati o mancanti.
- Test sicurezza per non esposizione dati sensibili in risposte ed errori.
- Validazione con Pydantic dei dati di input.
- Test di carico base per endpoint login e protezione endpoint critici.
- Copertura test integrata nel CI/CD pipeline.
- Piano esteso per test end-to-end e stress test in successivi cicli.

## 11. Rischi tecnici
- Contraddizione tecnologia iniziale: stack Java vs vincolo FastAPI da risolvere per non compromettere progetto.
- Gap competenze Python/FastAPI nel team, necessità formazione dedicata.
- Mancata implementazione completa di policy di scadenza e revoca token reset e JWT potrebbe esporre a rischi di sicurezza.
- Reset password base senza invio email limita usabilità e sicurezza reali.
- Assenza di meccanismi anti brute force e monitoraggio avanzato.
- Gestione sicura e storage segreti JWT e DB è critico.
- Mancanza di audit logging avanzato e gestione ruoli limita scalabilità enterprise.
- Nessuna protezione avanzata token reset (es. cifratura token in DB).
- Mancanza di test end-to-end nel MVP riduce confidence su stabilità.
- Possibile esposizione a vulnerabilità future senza policy di aggiornamento librerie.

## 12. Struttura file proposta
```
/app
  |-- main.py                   # Avvio FastAPI e montaggio router
  |-- config.py                 # Configurazioni e parametri ambiente
  |-- database.py               # Connessione e sessione PostgreSQL async
  |-- models.py                 # Definizione modello ORM (User, ResetToken)
  |-- schemas.py                # Schemi Pydantic per validazioni input/output
  |-- auth.py                   # Funzioni JWT, hashing password, dependency protettiva
  |-- services.py               # Logica business (registrazione, login, reset)
  |-- routes/
      |-- auth.py               # Endpoints autenticazione (/register, /login, /reset-password)
      |-- protected.py          # Endpoint protetto esempio (/protected)
  |-- utils.py                  # Funzioni utilitarie (token random, validazioni)
  |-- logger.py                 # Configurazione logging e audit
/tests
  |-- test_auth.py              # Unit e integration test auth, hashing, JWT
  |-- test_reset_password.py    # Test logica reset password e token
  |-- test_protected.py         # Test protezione endpoint
  |-- test_validation.py        # Test validazione input Pydantic
```

## 13. Piano di implementazione
1. Setup ambiente FastAPI con configurazione base e connessione sicura a PostgreSQL (database.py, config.py).
2. Definizione modelli dati (User, ResetToken) con restrizioni e indicizzazione.
3. Implementazione hashing e verifica password in auth.py; generazione e verifica JWT con configurazione segreti.
4. Sviluppo endpoint registrazione (/auth/register) con validazione email, salvataggio password hashata.
5. Sviluppo endpoint login (/auth/login) con generazione token JWT.
6. Implementazione dependency FastAPI per protezione endpoint con verifica token JWT.
7. Creazione endpoint protetto di esempio (/protected).
8. Implementazione reset password base:
   - richiesta reset token (/auth/reset-password/request) con salvataggio token e timestamp.
   - conferma reset cambio password (/auth/reset-password/confirm) con validazione token e invalidazione.
9. Integrazione logging e error handling centralizzato.
10. Documentazione automatica API tramite OpenAPI (FastAPI).
11. Sviluppo test unitari e di integrazione per tutte le funzionalità chiave.
12. Revisione sicurezza codice, policy gestione segreti e audit logging base.
13. Pianificazione formazione team per FastAPI e Python.
14. Consegna MVP con manuale d’uso e linee guida per policy futura su token e complessità password.