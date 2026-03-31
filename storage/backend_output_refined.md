MODULE: BACKEND VERSION: 1
## 1. Obiettivo backend
Progettare e implementare un backend API REST sicuro e scalabile per la gestione dell’autenticazione utenti, che consenta la registrazione, login con email e password, gestione token JWT per protezione endpoint, e una funzionalità base di reset password. Il sistema dovrà garantire la sicurezza dei dati sensibili (password e token) e offrire una API documentata tramite OpenAPI/Swagger, realizzata utilizzando FastAPI e PostgreSQL come database persistente.

## 2. Assunzioni tecniche
- Email univoca per utente, usata come identificativo principale.
- Password salvate in forma hashata con criteri minimi di sicurezza (minimo 8 caratteri, complessità base: alfanumerica e simboli).
- Token JWT generati e validati secondo best practice, contenenti user ID e data di scadenza esplicita.
- Gestione token JWT senza revoca o blacklist nel MVP, ma con possibile roadmap futura.
- Reset password tramite token/flag temporaneo memorizzato nel DB con scadenza definita; nessuno invio email integrato.
- Modulazione codice con separazione chiara tra modelli dati, business logic, API e autenticazione.
- Utilizzo di transazioni e metodi per evitare condizioni di gara alla registrazione email duplicata.
- Versioni minime consigliate: Python 3.8+, FastAPI 0.70+, PostgreSQL 12+, PyJWT o equivalente aggiornato.
- Compliance base GDPR con rispetto della privacy e conservazione dati minima necessaria, senza dati inutili.
- Logging critico sicuro per accessi e errori, evitando informazioni sensibili.
- Testing automatizzato per copertura flussi critici.

## 3. Architettura backend
Architettura a microservizio monolitico per autenticazione, costruita su FastAPI con pattern MVC-like modulare:
- Presentation layer: API REST con validazione request/response, documentazione OpenAPI.
- Service layer: logica applicativa astraente dalla persistenza ed endpoint, incluse regole di autenticazione e reset password.
- Data access layer: interazione con PostgreSQL tramite ORM SQLAlchemy o equivalente.
- Security layer: gestione hashing password (bcrypt o Argon2), generazione e verifica JWT, middleware o dipendenza FastAPI per protezione endpoint.
Le componenti sono disaccoppiate e riutilizzabili, con configurazioni centralizzate (es. segreti JWT, connessione DB).

## 4. Moduli e responsabilità
- models.py: definizione modelli dati User, ResetPasswordToken, schemi Pydantic per richieste/risposte.
- database.py: configurazione e creazione sessioni DB, gestione transazioni.
- auth.py: funzioni per hashing password, verifica credenziali, generazione/verifica JWT.
- dependencies.py: dipendenze FastAPI, inclusa quella di autenticazione JWT per protezione endpoint.
- api/authentication.py: endpoint registrazione, login, reset password (richiesta token e aggiornamento password).
- api/protected.py: endpoint demo protetti tramite autenticazione JWT.
- main.py: setup applicativo FastAPI, inclusione router e middleware.
- tests/: suite test unitari e integrazione per copertura flussi critici di autenticazione, accesso, reset password.
- config.py: configurazioni centralizzate (segreti, parametri sicurezza, scadenze token).

## 5. API principali
- POST /register: registrazione utente (email, password) con validazione e risposta 201 o errore 409 per email duplicata.
- POST /login: autenticazione utente, restituisce token JWT valido (con scadenza).
- GET /protected: endpoint test protetto da verifica JWT, risponde 200 solo se token valido.
- POST /reset-password/request: richiesta reset password, genera token/flag temporaneo associato utente.
- POST /reset-password/confirm: uso token reset validato per aggiornare password (con validazione password).
  
Errori standard HTTP:
- 400 Bad Request per dati non validi.
- 401 Unauthorized per token assente, scaduto o invalido.
- 409 Conflict per email duplicata.

## 6. Business logic
- Validazione input utente con criteri di base per email e password (min 8 caratteri, almeno una lettera e cifra).
- Hashing password con algoritmo sicuro (bcrypt o Argon2), comparazione tramite verifica hash.
- Alla registrazione: controllo email unica con locking per evitare race condition; salvataggio in transazione sicura.
- Login: verifica credenziali, generazione JWT contenente ‘sub’ con user ID e ‘exp’ per scadenza generica (es. 15-60 min).
- Middleware per estrazione e verifica JWT, rifiuto richieste senza o con token non validi.
- Reset password: generazione token univoco temporaneo (UUID con scadenza es 15 min) salvato in DB; conferma reset con token valido e aggiornamento hash password.
- Tutta la logica è asincrona dove possibile, per integrazione e scalabilità futura.

## 7. Persistenza e integrazioni
- Database PostgreSQL per persistenza utenti, hash password, e token reset.
- Tabelle principali:
   - users: id (PK), email (unique), hashed_password, data_creazione.
   - reset_password_tokens: id (PK), user_id (FK), token (univoco), expiration_timestamp, used_flag.
- ORM per gestione transazioni, mapping, e interrogazioni.
- Nessuna integrazione esterna necessaria (ad es. email), ma struttura predisposta per future estensioni.

## 8. Autenticazione e autorizzazione
- Password gestita con hashing sicuro con salt.
- JWT generati con chiave segreta configurabile, algoritmo HMAC SHA256 standard.
- Payload minimo contenente user_id (sub) e scadenza esplicita.
- Middleware FastAPI per estrazione token dal header Authorization “Bearer”, verifica validità/expiration.
- Endpoint protetti richiedono autenticazione JWT per accedere alle risorse.
- Reset password gestito tramite token temporaneo salvato nel DB con scadenza e flag usato/non usato per evitare riutilizzo.

## 9. Gestione errori
- Errori HTTP standardizzati con messaggi minimali per non esporre informazioni sensibili.
- 400 Bad Request per input errato o non conforme alle policy base.
- 401 Unauthorized per token JWT assente, invalido o scaduto.
- 404 Not Found per risorse non trovate in reset password o endpoint.
- 409 Conflict quando tentata registrazione con email già esistente (gestita transazionalmente).
- Logging sicuro con log errori backend in forma anonima per tracciabilità senza leak dati.
- Risposte JSON uniformi contenenti 'detail' per descrizione errore sintetica.

## 10. Strategia di test backend
- Test unitari per funzioni hashing password, verifica JWT, validazione dati.
- Test di integrazione per flussi completi:
  - Registrazione user con email valida, e tentativo duplicato.
  - Login con credenziali corrette e errate.
  - Accesso a endpoint protetti con token valido, scaduto e manomesso.
  - Reset password: richiesta token, uso token valido, tentativo uso token scaduto o già usato.
- Test di concorrenza per registrazione simultanea stessa email, verifica atomicità.
- Test di conformità output OpenAPI/Swagger e controllo documentazione automatica.
- Test sicurezza base su gestione token (es. manomissione, JWT forged).
- Coverage report periodico per garantire copertura requisiti critici.

## 11. Rischi tecnici
- Mancanza di revoca token JWT esplicita: eventuale token compromesso valido fino a scadenza.
- Reset password semplice senza validazione esterna (es. email), potenziale abuso se token leak.
- Politica password base può non garantire sicurezza elevata, rischio credenziali fragili.
- Gestione errori e logging non dettagliato per non compromettere privacy ma limita auditing.
- Gestione concorrente email duplicata complessa in contesti ad alto carico.
- Assenza di compliance GDPR dettagliata: minimizzazione dati e logica conservazione da definire meglio.
- Versioni software non rigide impongono aggiornamenti e testing continui per sicurezza.

## 12. Struttura file proposta
```
/app
 ├── main.py                      # Entry point FastAPI application
 ├── config.py                    # Configurazioni e variabili ambiente
 ├── database.py                  # Connessione e sessione PostgreSQL tramite ORM
 ├── models.py                    # ORM models: User, ResetPasswordToken
 ├── schemas.py                   # Pydantic schemi per request/response
 ├── auth.py                      # Gestione hashing, JWT e verifica credenziali
 ├── dependencies.py              # Dipendenze FastAPI per autenticazione e DB
 ├── api
 │    ├── authentication.py       # Endpoints register, login, reset password
 │    ├── protected.py            # Endpoint di prova protetti JWT
 ├── tests
 │    ├── test_auth.py            # Test unitari e integrazione autenticazione
 │    ├── test_reset_password.py  # Test reset password
 │    ├── test_protected.py       # Test endpoint protetti
 └── utils.py                     # Utilità comuni (e.g. gestione logging)
```

## 13. Piano di implementazione
1. Progettazione schema DB utenti e reset password token; creazione script migrazioni.
2. Setup progetto FastAPI con connessione DB PostgreSQL e ORM.
3. Implementazione modelli dati, schemi Pydantic e configurazioni ambiente.
4. Sviluppo funzionalità hashing password e gestione JWT.
5. Endpoint registrazione: validazione dati, verifica email unica con transazione, hashing password e salvataggio.
6. Endpoint login: verifica credenziali, generazione e restituzione token JWT.
7. Middleware/dipendenza di autenticazione per protezione endpoint.
8. Endpoint protetti di esempio (/protected) per validare controllo accesso tramite JWT.
9. Funzionalità reset password:
   - Endpoint richiesta reset con generazione token temporaneo.
   - Endpoint aggiornamento password con validazione token.
10. Logging sicuro e gestione errori centralizzata.
11. Sviluppo test unitari e integrazione coprendo i flussi principali.
12. Abilitazione documentazione OpenAPI/Swagger automatica con FastAPI.
13. Revisione codice, eventuali correzioni, e rilascio MVP.

Questa proposta rispetta la specifica PM e integra i requisiti emersi dal feedback QA, correggendo punti di debolezza sulla sicurezza password, gestione concorrenza email e reset password. Restano previste implementazioni future di revoca token e compliance GDPR più dettagliata.