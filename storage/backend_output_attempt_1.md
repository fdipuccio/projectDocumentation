MODULE: BACKEND VERSION: 1
## 1. Obiettivo backend
Sviluppare un backend API REST basato su FastAPI che consenta la gestione sicura dell’autenticazione utenti tramite email e password, inclusi i flussi di registrazione, login con generazione di token JWT, protezione di endpoint riservati e una funzionalità base di reset password. Il sistema deve garantire persistenza dati su PostgreSQL, implementare best practice di sicurezza per la gestione delle credenziali e dei token, e fornire documentazione API chiara e completa.

## 2. Assunzioni tecniche
- Il backend è sviluppato in Python mediante FastAPI, in linea con i vincoli tecnici obbligatori.
- PostgreSQL è il database di persistenza unico e centrale per la memorizzazione dati utenti.
- Le password sono gestite con hash sicuro (ad esempio Argon2 o bcrypt) e salting.
- JWT sono utilizzati come token di autenticazione, firmati con segreto e algoritmo standard (es. HS256), con scadenza configurabile.
- Il flusso di reset password prevede la generazione e memorizzazione di un token reset, che il client può usare per autorizzare il cambio password; invio email è fuori scope.
- Modello utente base: email unica, hash password, token reset password, data creazione/modifica.
- Il backend espone API REST documentate da OpenAPI (Swagger) generata da FastAPI.
- Nessuna implementazione frontend o integrazioni esterne sono previste.
  
## 3. Architettura backend
Architettura monolitica basata su FastAPI che espone un set di endpoint REST HTTP.  
- Livello API: gestione delle richieste HTTP, validazione input/output, autenticazione tramite dipendenze FastAPI.  
- Business Logic: servizi per gestione utenti, autenticazione, token JWT, reset password.  
- Persistenza: interfaccia con database PostgreSQL tramite ORM (es. SQLAlchemy/asyncpg).  
- Configurazione centralizzata per segreti e parametri sicurezza (es. chiave JWT, durata token).  
- Logging e gestione errori per monitoraggio e tracciamento.  
- Test unitari integrati per copertura funzioni core.

## 4. Moduli e responsabilità
- **app.main**: entry point FastAPI, configurazione endpoints e middleware.  
- **app.models**: definizione modello dati utente e DTO (Data Transfer Object).  
- **app.database**: setup connessione e gestione sessioni DB PostgreSQL.  
- **app.schemas**: definizione Pydantic schema per validazione input/output richieste API.  
- **app.auth**: gestione autenticazione, generazione e verifica JWT, hashing password.  
- **app.api.v1.routes**: endpoint REST divisi per funzionalità (registrazione, login, reset password, endpoint protetto).  
- **app.services**: logica di business per utenti, login, reset password.  
- **app.utils**: funzioni di supporto per sicurezza, hashing, validazioni.  
- **app.tests**: test unitari per singole funzionalità chiave.

## 5. API principali
- **POST /auth/register**  
  Permette la registrazione di un nuovo utente con email e password. Valida input, esegue hashing e salva utente.  
- **POST /auth/login**  
  Verifica credenziali email/password, restituisce token JWT firmato con claims standard (user_id, exp).  
- **POST /auth/reset-password/request**  
  Accetta richiesta reset password via email, genera token di reset associato all’utente (persistente).  
- **POST /auth/reset-password/confirm**  
  Accetta token reset e nuova password, valida token, aggiorna password hashed in DB e invalida token.  
- **GET /protected**  
  Endpoint di test protetto, accessibile solo con JWT valido. Restituisce dati di esempio per verifica autenticazione.

## 6. Business logic
- Verifica univocità email alla registrazione.  
- Hash sicuro delle password con salting (Argon2 o bcrypt).  
- Gestione sicura e cifrata dei token JWT: segreto in config, algoritmo HS256, durata token configurabile (es. 15-60 minuti).  
- Token reset password: stringa casuale e univoca generata criptograficamente, associata all’utente con timestamp per eventuale controllo interno (anche se scadenza esplicita non richiesta).  
- Validazione token JWT in dependencies FastAPI per protezione endpoint.  
- Protezione da accessi senza autenticazione o token non valido mediante HTTP 401 Unauthorized.  
- Nessuna esposizione di password o dati sensibili nei payload o log.  
- Gestione errori business standard (utente esistente, credenziali errate, token non valido).

## 7. Persistenza e integrazioni
- PostgreSQL tramite SQLAlchemy (consigliato in modalità asincrona con asyncpg) per gestione connessioni e sessioni.  
- Modello utente in DB con campi minimi: id (UUID o serial), email unique, password_hash, reset_token, timestamps.  
- Nessuna integrazione esterna (es. SMTP, MFA).  
- Trattamento dati secondo best practice sicurezza dati sensibili.

## 8. Autenticazione e autorizzazione
- JWT come meccanismo di autenticazione principale, generati e firmati sul login.  
- Segreto JWT configurabile tramite variabili ambiente.  
- Authentication dependency FastAPI che valida header Authorization “Bearer <token>”, decodifica, verifica firma e scadenza.  
- Endpoint protetti si appoggiano a dependency per controllare autenticazione.  
- Nessuna gestione ruoli o autorizzazioni complesse, solo presenza token valido garantisce accesso.

## 9. Gestione errori
- Risposte API con codici HTTP appropriati (400 per input invalidi, 401 per errori autenticazione, 409 per conflitti come email duplicata, 500 per errori interni).  
- Messaggi di errore chiari ma non esposti dati sensibili.  
- Log degli errori server per monitoraggio e debug.  
- Validazione input tramite Pydantic genera errori con dettagli precisi su campi errati.

## 10. Strategia di test backend
- Test unitari per funzioni chiave: hashing password, generazione/verifica JWT, validazione token reset.  
- Test integrazione per endpoint API principali (registrazione, login, reset password, endpoint protetto) usando client test FastAPI.  
- Simulazione casi di errore (credenziali errate, token scaduto/non valido, email duplicata).  
- Coverage focalizzata su sicurezza e corretto flusso autenticazione.

## 11. Rischi tecnici
- Potenziale gap di competenze FastAPI nel team considerando stack Java originario.  
- Mancanza di politica formale per scadenza e revoca token reset password e JWT (potrebbe rischiare sicurezza).  
- Limiti del reset password "base" senza integrazione con email o meccanismi di verifica.  
- Contraddizione stack Java vs obbligo FastAPI/Python: necessita conferma e allineamento.  
- Possibili vulnerabilità se gestione segreti JWT non sicura o errata esposizione dati sensibili.

## 12. Struttura file proposta
```
app/
 ├─ main.py                 # Entrypoint FastAPI
 ├─ models.py               # Modello dati DB utenti
 ├─ database.py             # Connessione PostgreSQL
 ├─ schemas.py              # Pydantic schemi request/response
 ├─ auth.py                 # Funzioni autenticazione, hashing e JWT
 ├─ services/
 │   ├─ user_service.py     # Logica registrazione, login, reset password
 ├─ api/
 │   ├─ v1/
 │   │   ├─ routes.py       # Definizione endpoint API
 ├─ utils.py                # Funzioni di supporto (es. generazione token)
 ├─ tests/                  # Test unitari e integrazione
 │   ├─ test_auth.py
 │   ├─ test_user.py
 config.py                  # Configurazioni (JWT secret, db url)
requirements.txt            # Dipendenze progetto
```

## 13. Piano di implementazione
1. Setup ambiente FastAPI con virtualenv e dipendenze (FastAPI, Uvicorn, SQLAlchemy/asyncpg, passlib, PyJWT).  
2. Configurazione connessione sicura a PostgreSQL (database.py).  
3. Definizione modello dati utente (models.py) e schema Pydantic (schemas.py).  
4. Implementazione hashing password e funzioni JWT (auth.py).  
5. Endpoint registrazione (POST /auth/register) con validazione e persistenza.  
6. Endpoint login (POST /auth/login) con verifica credenziali, generazione e risposta JWT.  
7. Configurazione dependency FastAPI per protezione endpoint con JWT.  
8. Endpoint protetto di test (GET /protected).  
9. Flusso reset password base: generazione token (POST /auth/reset-password/request), salvataggio token, cambio password con verifica token (POST /auth/reset-password/confirm).  
10. Documentazione automatica con OpenAPI/Swagger generata da FastAPI.  
11. Scrittura test unitari e integrazione.  
12. Refactoring e revisione codice per sicurezza e pulizia.  
13. Documentazione tecnica finale API e istruzioni di deployment.