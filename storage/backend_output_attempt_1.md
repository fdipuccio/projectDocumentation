MODULE: BACKEND VERSION: 1
## 1. Obiettivo backend
Sviluppare un backend API REST con FastAPI per la gestione dell’autenticazione utenti, includendo la registrazione, il login tramite email e password, l’emissione di token JWT per sessioni sicure, la protezione di endpoint tramite autenticazione JWT e una funzionalità di reset password base. La soluzione deve garantire persistenza dei dati su PostgreSQL ed esporre API documentate con OpenAPI/Swagger.

## 2. Assunzioni tecniche
- Backend implementato esclusivamente con FastAPI (Python).
- Database PostgreSQL usato per persistenza utenti e dati di autenticazione.
- Token JWT generati e verificati secondo standard (HS256 o RS256), senza personalizzazioni.
- Password hashate con algoritmo sicuro (es. bcrypt) senza requisiti speciali di policy o lockout.
- Reset password gestito tramite token temporanei memorizzati e validati lato backend, senza integrazione con sistemi email/SMS.
- No gestione ruoli o permessi oltre alla semplice autenticazione.
- Documentazione API generata automaticamente da FastAPI.
- Configurazioni di rete, sicurezza e deploy esterne all’ambito MVP.

## 3. Architettura backend
Architettura monolitica orientata a servizi REST, basata su FastAPI:
- Layer API REST per esposizione endpoint.
- Layer di business logic per trattamento regole applicative (es. validazione dati, gestione token).
- Layer di persistenza con accesso a PostgreSQL tramite ORM (es. SQLAlchemy).
- Middleware/dependency FastAPI per gestione autenticazione e protezione endpoint.
- Componente dedicato per gestione JWT (creazione, firma, verifica).
Interazione sincrona diretta tra client e API, senza sistemi di messaggistica o code esterne.

## 4. Moduli e responsabilità
- Modulo Users: gestione modello utente, registrazione, verifica credenziali.
- Modulo Auth: login, generazione token JWT, verifica token.
- Modulo PasswordReset: creazione e validazione token reset password.
- Modulo API Endpoints: definizione API REST per registrazione, login, reset password, endpoint protetti.
- Modulo Database: configurazione e connessione PostgreSQL, accesso dati.
- Middleware/Auth Dependency: protezione endpoint via verifica JWT.
- Documentazione API: esposizione schema OpenAPI automatico.
- Test: suite per unit e integrazione.

## 5. API principali
- POST /register  
  Input: email, password  
  Output: conferma registrazione o errore (es. email già in uso)

- POST /login  
  Input: email, password  
  Output: token JWT (con scadenza standard configurata) o errore di credenziali

- GET /protected-example  
  Accesso: JWT valido richiesto  
  Output: risposta di test autenticazione (es. dati utente o messaggio)

- POST /reset-password/request  
  Input: email  
  Output: conferma generazione token reset (senza invio email)

- POST /reset-password/confirm  
  Input: token reset, nuova password  
  Output: conferma reset password o errore (token non valido o scaduto)

## 6. Business logic
- Validazione input per registrazione (email formattata correttamente, password non vuota).
- Controllo univocità email su registrazione.
- Hash sicuro della password al salvataggio.
- Verifica credenziali al login confrontando hash password.
- Generazione JWT con claim minimo (id utente, expiration time).
- Protezione endpoint tramite verifica token JWT in header Authorization.
- Generazione token reset password con scadenza breve, memorizzazione nel DB legata all’utente.
- Validazione token reset e aggiornamento password hashata.

## 7. Persistenza e integrazioni
- PostgreSQL per salvataggio dati utenti (id, email, password hash, data creazione).
- Tabella dedicata per token di reset password con relazione utente, token, timestamp di scadenza.
- Utilizzo ORM SQLAlchemy (o equivalente) per manipolazione dati.
- Nessuna integrazione esterna prevista (es. email o notifiche).
- Connessione DB gestita tramite pool e configurazione standard di FastAPI.

## 8. Autenticazione e autorizzazione
- Autenticazione basata su token JWT firmati (algoritmo HS256 o RS256).
- Token JWT generato al login, con scadenza configurabile (es. 15-60 minuti).
- Estrazione token dall’header Authorization Bearer.
- Middleware FastAPI o dependency injection per verifica automa token e recupero utente corrente.
- Nessuna autorizzazione avanzata: accesso endpoint protetti solo se autenticati.

## 9. Gestione errori
- Errori di input (es. email non valida, password mancante) con risposta 400 Bad Request e messaggi chiari.
- Errori di autenticazione (es. credenziali errate, token mancante o invalido) con 401 Unauthorized.
- Errori reset password (es. token scaduto, token inesistente) con 400 Bad Request o 401 se necessario.
- Gestione duplicati email su registrazione con errore 409 Conflict.
- Log minimali interni per sviluppo, esclusa gestione centralizzata.
- Risposte API con struttura JSON coerente per errori e successi.

## 10. Strategia di test backend
- Test unitari per moduli business (es. hashing, generazione token, validazione dati).
- Test di integrazione per endpoint HTTP, con simulazione DB test PostgreSQL.
- Copertura test su flusso registrazione, login, accesso protetto, reset password.
- Test su condizioni di errore (input non valido, token invalidi).
- Possibile uso di pytest con fixture FastAPI app e dipendenze DB mock.
- Esecuzione test automatica con comando standard prima del deploy.

## 11. Rischi tecnici
- Ambiguità nello stack tecnologico: FastAPI obbligatorio ma esiste ambiente Java REST API esistente; necessaria chiarezza su coexistence o migrazione.
- Dettaglio limitato sulle politiche di sicurezza token (durata, refresh non implementato).
- Mancanza di gestione di utenti duplicati oltre all’errore esplicito.
- Assenza di meccanismi anti-brute force o lockout password.
- Reset password senza invio email limita usabilità reale, da integrare in futuro.
- Gestione segreti (chiave JWT) e configurazioni sensibili da proteggere adeguatamente in produzione.

## 12. Struttura file proposta
```
/app
  /api
    auth.py             # endpoint login, register
    password_reset.py   # endpoint reset password
    protected.py        # esempio endpoint protetto
  /core
    config.py           # configurazioni applicativo (db, jwt)
    security.py         # funzioni hash, jwt
  /db
    base.py             # sessione e connessione db
    models.py           # modelli ORM utenti, reset token
    crud.py             # operazioni db su utenti, reset
  /schemas
    user.py             # Pydantic schemas input/output
    auth.py
  /tests
    test_auth.py
    test_password_reset.py
    test_protected.py
main.py                 # FastAPI app entry point
```

## 13. Piano di implementazione
1. Setup ambiente FastAPI e connessione PostgreSQL (definizione config e sessione DB).
2. Definizione modelli dati utenti e tabella token reset con ORM.
3. Implementazione endpoint registrazione con validazione e persistenza dati.
4. Implementazione endpoint login: verifica password, generazione token JWT.
5. Configurazione middleware/dependency per protezione endpoint con JWT.
6. Creazione endpoint protetto di esempio per test autenticazione.
7. Implementazione endpoint reset password: generazione e validazione token.
8. Scrittura test unitari e integrazione sulle funzionalità sviluppate.
9. Generazione e verifica documentazione API tramite schema FastAPI.
10. Revisione sicurezza base: hashing password, gestione segreti JWT.