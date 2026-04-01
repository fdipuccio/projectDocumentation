MODULE: BACKEND VERSION: 1
## 1. Obiettivo backend
Progettare e implementare un backend API RESTful per la gestione dell’autenticazione utenti che consenta la registrazione tramite email e password, il login con generazione di token JWT per sessioni sicure, la protezione di endpoint riservati tramite validazione JWT e una funzionalità base per il reset password tramite token. La soluzione dovrà essere sviluppata con FastAPI, utilizzando PostgreSQL come database di persistenza e rispettare le best practice di sicurezza minimali per la gestione delle credenziali e dei token.

## 2. Assunzioni tecniche
- Password degli utenti saranno salvate in forma hashed utilizzando algoritmi sicuri (es. bcrypt o Argon2).
- La chiave segreta per la generazione e verifica dei token JWT sarà configurata in ambiente, con scadenza standard del token (es. 60 minuti).
- Nel reset password, il token di reset sarà un campo salvato nel database lato utente, con gestione di scadenza tramite timestamp.
- Non sarà gestito l’invio email per reset password né sistemi avanzati di sicurezza (MFA, rate limiting).
- PostgreSQL è già predisposto e accessibile con credenziali corrette.
- L’API sarà stateless, con gestione sessioni esclusivamente tramite JWT.
- La documentazione sarà generata automaticamente tramite OpenAPI integrata FastAPI.
- I test automatici copriranno i casi principali di registrazione, login, reset password e accesso protetto.
  
## 3. Architettura backend
Architettura a 3 livelli: 
- Livello API / Web framework con FastAPI esposto via HTTP REST, gestisce routing, validazione input/output e integrazione dipendenze.
- Livello di business logic per la gestione dell’autenticazione, hashing password, generazione token, verifiche di sicurezza, generazione token reset.
- Livello persistence con repository su PostgreSQL tramite ORM (preferibilmente SQLAlchemy o equivalente), con tabelle per utenti con campi essenziali identificativi, password hash e token reset.
L’architettura prevede middleware o dependency injection FastAPI per la verifica JWT e protezione endpoint.

## 4. Moduli e responsabilità
- **modello_user.py:** Definizione modello dati utente (email, password_hash, reset_token, reset_expiry).
- **database.py:** Configurazione e connessione al database PostgreSQL.
- **schemas.py:** Pydantic model per validazione request/response (registrazione, login, reset password).
- **auth.py:** Funzioni di hashing password, generazione e verifica token JWT, generazione token reset password.
- **dependencies.py:** Funzioni FastAPI Depends per autenticazione JWT e recupero utente autenticato.
- **routes_auth.py:** Endpoint API per registrazione, login, reset password.
- **routes_protected.py:** Endpoint esempio protetto che richiede autenticazione.
- **main.py:** Setup FastAPI app, caricamento router, configurazioni generali.
- **tests/:** Test automatici con casi di registrazione, login, reset e accesso protetto.
  
## 5. API principali
- `POST /register`: registra un nuovo utente con email e password; valida formato email e complessità password minima, salva utente con password hashed.
- `POST /login`: riceve email e password, verifica credenziali, restituisce token JWT con payload minimo (es. user_id, email).
- `POST /password-reset/request`: riceve email; se utente esiste, genera token reset e lo memorizza con scadenza nel db (il token viene restituito in risposta, no invio email).
- `POST /password-reset/confirm`: riceve token reset + nuova password; valida token, aggiorna password con hash e rimuove token reset.
- `GET /protected/example`: esempio di endpoint protetto; accessibile solo con header Authorization: Bearer <JWT> valido.

## 6. Business logic
- Registrazione: validazione dati, hashing password con bcrypt/Argon2, verifica univocità email, salvataggio record.
- Login: verifica esistenza utente, confronto password usando hash, generazione JWT firmato tramite chiave segreta backend, con payload minimale (user_id, email) e scadenza.
- Protezione endpoint tramite dependency che decodifica JWT, verifica firma e scadenza, recupera utente nel db.
- Reset password: generazione token unico (UUID o secure random), salvataggio token + scadenza temporale in db. Nel confirm reset, validazione token e scadenza, update password hashed e reset token rimosso.
- Nessuna gestione avanzata di sessioni o invalidazione token posta login.

## 7. Persistenza e integrazioni
- PostgreSQL come unico sistema di persistenza.
- Tabella utenti con i seguenti campi minimi: id (pk), email (unique), password_hash, reset_token (nullable), reset_token_expiry (nullable, timestamp).
- I dati persistiti saranno coerenti con best practice sicurezza: nessuna password in chiaro, token reset sufficientemente casuale e temporaneamente valido.
- Nessuna integrazione esterna per email o altri servizi.

## 8. Autenticazione e autorizzazione
- Autenticazione stateless con JSON Web Token (JWT) firmati con chiave segreta backend.
- Token contenente claims essenziali: user_id, email, exp (scadenza).
- Protezione endpoint tramite dipendenza FastAPI che controlla header Authorization, verifica token JWT e carica entità utente.
- Accesso negato con HTTP 401 Unauthorized se token assente, scaduto o non valido.
- Nessun livello aggiuntivo di autorizzazione (ruoli, permessi).

## 9. Gestione errori
- Risposte API coerenti e aderenti a standard REST con codici: 
  - 400 Bad Request per errori di validazione input (es. email non valida, password debole).
  - 401 Unauthorized per errori di autenticazione (login fallito, token JWT invalido).
  - 404 Not Found per risorse non trovate (es. utente reset password non esistente o token reset non valido).
  - 500 Internal Server Error per errori imprevisti lato server.
- Messaggi di errore sintetici e non espongono dettagli sensibili.
- Validazione input tramite Pydantic.
- Logging degli errori a livello applicativo per debug e auditing.

## 10. Strategia di test backend
- Test automatici unitari e funzionali con framework standard (pytest).
- Casi di test:
  - Registrazione: successo, email duplicata, dati invalidi.
  - Login: corretto, password errata, utente non esistente.
  - Protezione endpoint: accesso con e senza token valido.
  - Reset password: richiesta token, cambio password con token valido e token scaduto o invalido.
- Mock isolati per testare la logica senza dipendenze esterne ove possibile.
- Test di integrazione con db PostgreSQL in ambiente di test.

## 11. Rischi tecnici
- Definizione incompleta del formato e durata token JWT può portare a inconsistenze o problemi di sicurezza.
- Reset password gestito solo via token memorizzato db, senza verifica tramite email può creare rischio di uso improprio.
- Mancanza di protezioni da attacchi comuni (rate limiting, brute force) espone la soluzione a rischi in produzione.
- Nessuna gestione della revoca di token JWT: logout o invalidazioni non previste.
- Possibile difficoltà di scalabilità della soluzione nel carico elevato o in contesti distribuiti (ad esempio token reset memorizzati solo nel db).
- Dipendenza dalla corretta configurazione della chiave segreta per JWT; gestione della rotazione non prevista.

## 12. Struttura file proposta
```
backend/
│
├── main.py                         # Entry point FastAPI app
├── database.py                     # Connessione e configurazione DB
├── models/
│   └── user.py                    # Modello ORM User
├── schemas/
│   └── user.py                    # Pydantic schemi input/output
├── auth/
│   ├── auth_utils.py              # Hash password, JWT, reset token
│   └── dependencies.py            # Depends FastAPI per auth
├── routes/
│   ├── auth_routes.py             # Registrazione, login, reset password
│   └── protected_routes.py        # Endpoint protetti di esempio
├── tests/
│   ├── test_auth.py               # Test registrazione e login
│   ├── test_reset_password.py     # Test reset password
│   └── test_protected.py          # Test accesso endpoint protetti
├── config.py                      # Configurazioni, segreti ambiente
└── utils.py                       # Eventuali utility generali
```

## 13. Piano di implementazione
1. Definire e implementare schema utenti con campi necessari (email, password_hash, reset_token, token expiry).
2. Configurare ambiente FastAPI e connettere a PostgreSQL con test di connessione.
3. Implementare endpoint `/register` con validazione e hashing password.
4. Implementare endpoint `/login` con controllo credenziali e generazione JWT.
5. Creare dipendenza FastAPI per verifica token JWT e recupero utente autenticato.
6. Realizzare endpoint protetto di esempio `/protected/example` utilizzando la dipendenza di autenticazione.
7. Sviluppare funzionalità base reset password:
    - `/password-reset/request` che genera token reset e lo salva in db.
    - `/password-reset/confirm` che consente cambio password con token reset valido.
8. Scrivere test automatici per tutti gli endpoint principali e logica.
9. Configurare e verificare documentazione OpenAPI generata automaticamente da FastAPI.
10. Verificare qualità e sicurezza della soluzione e predisporre ambiente per deploy.