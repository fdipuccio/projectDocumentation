MODULE: BACKEND VERSION: 1
## 1. Obiettivo backend
Progettare e sviluppare un backend API sicuro, scalabile e conforme ai requisiti business per la gestione dell’autenticazione utenti. Le funzionalità principali includono la registrazione di utenti con email univoca e password protette, login con generazione di token JWT per gestione sessioni, protezione degli endpoint riservati tramite validazione JWT, e una funzionalità base di reset password tramite email contenente link/token temporaneo. Il backend sarà sviluppato con FastAPI, utilizzando PostgreSQL per la persistenza dati, in Python.

## 2. Assunzioni tecniche
- Gli utenti dispongono di una email univoca e password che saranno validate secondo policy minima da definire (es. lunghezza minima e complessità base).
- Il sistema di reset password invierà email tramite un’integrazione SMTP o placeholder configurabile, senza template avanzati.
- Il JWT emesso conterrà informazioni standard (es. user_id, expiry) senza gestione di revoca o blacklist.
- Non sono previsti ruoli o autorizzazioni granulari; la protezione endpoint è basata solo su autenticazione tramite token JWT valido.
- Persistenza dati utenti e token di reset sarà garantita da PostgreSQL, con schema utenti semplificato (email, password hash, token reset temporanei).
- Validazione input e gestione errori rispetterà standard RESTful e best practice di sicurezza basilari.
- Non sono previste features avanzate di sicurezza quali 2FA, CAPTCHA o limitazioni tentativi login.

## 3. Architettura backend
Architettura monolitica API REST basata su FastAPI che espone endpoint HTTP JSON per le operazioni di autenticazione e gestione utenti. Il backend mantiene una connessione persistente e asincrona a PostgreSQL tramite ORM o driver SQLAlchemy. L’intercettazione e validazione token JWT per i path protetti avviene mediante dipendenza FastAPI dedicata. La generazione e validazione token JWT sarà realizzata con librerie consolidate come PyJWT o equivalente. Lo scambio email per reset password sarà delegato a un componente SMTP configurabile. Il backend garantisce separazione nette tra controller (endpoint), logica business, accesso DB e sicurezza.

## 4. Moduli e responsabilità
- **Model**: Definizione schema dati utenti (email, password hash, campo reset token), tabelle PostgreSQL.
- **Database**: Configurazione e gestione connessione a PostgreSQL, operazioni CRUD utenti e token reset.
- **Auth**: Funzioni per hashing password (bcrypt), generazione e verifica token JWT, validazione input credenziali.
- **API/User**: Endpoint di registrazione, login, reset password (richiesta e modifica), endpoint di test protetto.
- **Security**: Middleware o dipendenza FastAPI per protezione endpoint mediante verifica token JWT valido.
- **Email**: Servizio per invio email reset password via SMTP o mock placeholder.
- **Utils**: Funzioni di supporto per validazione password, generazione token reset temporanei.
- **Error Handling**: Gestione e standardizzazione degli errori HTTP e risposte RESTful.

## 5. API principali
- `POST /register`: Registrazione utente; input email e password; controllo email unica, hashing password; risposta 201 o errore.
- `POST /login`: Autenticazione; input email e password; verifica credenziali; risposta con token JWT o errore 401.
- `GET /protected-example`: Endpoint esempio protetto; accesso solo con token JWT valido; risponde dati demo o 401.
- `POST /password-reset/request`: Invio email reset password; input email registrata; generazione token reset temporaneo; risposta 200 o errore 404.
- `POST /password-reset/confirm`: Modifica password; input nuovo password + token reset; verifica token valido, update password hash; risposta 200 o errore 400/401.

## 6. Business logic
- Validazione formale input (email formato corretto, password livelli minimi).
- Hashing password con bcrypt per salvataggio sicuro.
- Creazione token JWT firmato con chiave segreta, definendo scadenza standard (es. 15-60 min).
- Generazione token reset password randomici con scadenza breve (es. 1 ora) salvati in DB associati a user.
- Invio email con link contenente token reset (es. URL frontend placeholder con token).
- Verifica token reset validità al momento cambio password.
- Controllo univocità email in fase di registrazione, risposta esplicita di conflitto.
- Protezione dei parametri sensibili nelle risposte API.

## 7. Persistenza e integrazioni
- PostgreSQL come database primario, tabella `users` (id PK, email unique, password_hash, reset_token, reset_token_expiry).
- Salvataggio persistente e aggiornamento password e token reset.
- Connessione gestita tramite ORM SQLAlchemy o async driver compatibile con FastAPI.
- Servizio SMTP configurabile per invio email di reset password (possibile implementazione adattabile a servizi esterni).
- Nessuna integrazione diretta con provider esterni o sistemi terzi per autenticazione o mailing avanzato.

## 8. Autenticazione e autorizzazione
- Autenticazione tramite email e password standard.
- Generazione token JWT con payload contenente identificativo utente e scadenza.
- Middleware/dipendenza FastAPI verifica presenza e validità token JWT in header `Authorization: Bearer`.
- Accesso ai servizi protetti permesso solo se token valido; altrimenti risposta 401 Unauthorized.
- Non prevista autorizzazione di ruoli o permessi specifici.

## 9. Gestione errori
- Schema standardizzato di risposta errori RESTful con codici HTTP appropriati (400 Bad Request, 401 Unauthorized, 404 Not Found, 409 Conflict, 500 Internal Server Error).
- Validazione input con messaggi chiari per errori di formato o dati mancanti.
- Gestione collisione email registrata con risposta 409.
- Cattura e log errori non gestiti per debugging.
- Risposte JSON uniformi con struttura `{ "detail": "descrizione errore" }`.
- Prevenzione leak di informazioni sensibili nei messaggi.

## 10. Strategia di test backend
- Test unitari per funzioni di hashing password, generazione/verifica token JWT, validazione input.
- Test integrazione endpoint registrazione, login, reset password con casi positivi e negativi.
- Verifica meccanismo protezione endpoint protetti (accesso con/o senza token, token scaduti o invalidi).
- Test gestione errori e risposte HTTP.
- Esecuzione test automatizzata localmente o in pipeline CI.
- Possibile mock o stubbing per servizio invio email durante test.
- Uso OpenAPI/Swagger per validazione manuale endpoint.

## 11. Rischi tecnici
- Metodo di invio email per reset password non definito (potenziale complessità o mancata consegna).
- Mancanza di politiche di sicurezza definite per password e reset può causare vulnerabilità.
- Assenza di revoca token JWT limita gestione sicurezza sessioni.
- Gestione scadenza e rinnovo token può introdurre rischi se non calibrata bene.
- Possibili colli di bottiglia dovuti a connessioni DB mal ottimizzate (non parte del requisito però).
- Futuri requisiti di ruoli o autorizzazioni potrebbero richiedere refactoring.

## 12. Struttura file proposta
```
/app
  /api
    user.py               # endpoint register, login, reset password, protected routes
  /core
    config.py             # configurazioni app e DB
    security.py           # hashing, JWT generation/validation
    email_service.py      # servizio invio email reset password
  /db
    base.py               # setup connessione DB, sessioni
    models.py             # definizione modello User
    crud.py               # operazioni DB CRUD su utenti e token reset
  /schemas
    user.py               # Pydantic schemi input/output
  /utils
    validators.py         # validazione password, email, token reset
  main.py                 # entry point FastAPI app e registrazione route, dipendenze middleware
/tests
  test_auth.py            # test funzioni auth e endpoint
  test_reset_password.py  # test reset password
README.md
requirements.txt
```

## 13. Piano di implementazione
1. Definizione schema dati utenti PostgreSQL e tabella `users`.
2. Setup progetto FastAPI, connessione DB PostgreSQL, configurazione ambiente.
3. Sviluppo endpoint registrazione utente con hashing password bcrypt e validazione email unica.
4. Implementazione login con verifica credenziali e generazione token JWT.
5. Creazione middleware/dipendenza FastAPI per validazione cookie header Authorization e protezione endpoint.
6. Realizzazione endpoint protetto di test per verifica accesso autenticato.
7. Implementazione funzionalità reset password:
   - Generazione token reset temporaneo salvato DB.
   - Endpoint invio email reset password con token.
   - Endpoint modifica password dopo verifica token.
8. Stesura e integrazione test unitari e di integrazione per tutti gli endpoint e servizi.
9. Documentazione e verifica API usando OpenAPI/Swagger generato automaticamente da FastAPI.
10. Validazione finale, gestione errori, rifinitura dettagli sicurezza e best practice nella codifica.