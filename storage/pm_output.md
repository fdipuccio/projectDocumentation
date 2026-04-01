MODULE: PM VERSION: 1
## 1. Obiettivo
Svilupare un backend per un sistema di autenticazione utenti che consenta la registrazione, il login tramite email e password, la generazione di token JWT, la protezione di endpoint riservati e una funzionalità base di reset password.

## 2. Contesto e vincoli
- Il backend deve essere sviluppato utilizzando il framework FastAPI (Python).
- Il database di persistenza è PostgreSQL.
- Le funzionalità devono includere registrazione, login, generazione token JWT, protezione endpoint e reset password.
- Il token di autenticazione deve essere JWT.
- Stack attuale indicato include Java, REST API, PostgreSQL, JWT; tuttavia appare in contrasto con il vincolo tecnico esplicitamente richiesto di usare FastAPI (Python).
  
Vincoli reali:
- Uso obbligatorio di FastAPI per il backend.
- Uso obbligatorio di PostgreSQL come DB.

Vincoli evidenti:
- JWT per autenticazione.
- Endpoints protetti in base all'autenticazione.

## 3. Assunzioni
- Si assume che il team di sviluppo abbia competenze su FastAPI e Python, nonostante nello stack sia citato Java.
- Si assume che la gestione delle password includa salting e hashing sicuri.
- Il reset password "base" intende un semplice flusso di generazione token di reset e cambio password senza funzionalità avanzate come link con scadenza, captcha o verifica MFA.
- Non è specificata la modalità di invio dei token per il reset (es. email) né la configurazione SMTP, si assume che la parte invio email sia out of scope o da definire.
- Non è chiarito il modello utenti né gli attributi richiesti, si assume supporto minimo tramite email e password.

## 4. Scope MVP
- Endpoint di registrazione utente con email e password.
- Endpoint di login che restituisce token JWT valido.
- Meccanismo generazione e firma dei JWT con claims standard (es. user id, scadenza).
- Protezione di almeno un endpoint di esempio che richiede autenticazione JWT.
- Endpoint base per reset password: accettazione richiesta reset, generazione token reset, cambio password tramite token.
- Persistenza utenti in PostgreSQL.
- Implementazione secondo best practice di sicurezza di FastAPI e JWT.

## 5. Out of scope
- Frontend o interfacce grafiche.
- Implementazioni avanzate di reset password (es. gestione link email, notifiche).
- Integrazione con servizi di terze parti per invio email o MFA.
- Gestione ruoli o autorizzazioni complesse oltre l’autenticazione base.
- Testing end to end o deployment.

## 6. Task tecnici ordinati
1. Setup ambiente FastAPI con configurazione base.
2. Definizione modello dati utenti su PostgreSQL (email, password hash, reset token, etc.).
3. Implementazione registrazione utente con validazione email e hashing password.
4. Implementazione login con verifica credenziali e generazione JWT.
5. Configurazione JWT (segreto, algoritmo, durata token).
6. Implementazione middleware/dependencies FastAPI per protezione endpoint tramite JWT.
7. Sviluppo endpoint protetto di test.
8. Implementazione base reset password: accettazione richiesta reset, salvataggio token reset, endpoint cambio password con verifica token.
9. Setup connessione sicura a PostgreSQL.
10. Documentazione API con OpenAPI (Swagger) generata da FastAPI.
11. Testing unitari su funzioni chiave (es. autenticazione, token).

## 7. Acceptance criteria
- Registrazione crea un utente persistente con password hashed.
- Login con credenziali corrette restituisce token JWT firmato e valido.
- Accesso a endpoint protetto senza token o con token non valido è negato (401).
- Endpoint reset password consente l’inizializzazione cambio password tramite token.
- Tutte le funzionalità rispettano i vincoli tecnici (FastAPI, PostgreSQL).
- Codice documentato e API documentate mediante OpenAPI.
- Implementazione sicura senza esposizione di password o dati sensibili in chiaro.

## 8. Rischi e punti aperti
- Contraddizione tra stack segnalato (Java) e vincolo FastAPI (Python), necessita chiarire tecnologia da usare.
- Mancanza di dettaglio su flusso e sicurezza reset password (es. invio token, gestione scadenza).
- Assenza di requisiti su criteri password, politiche di validazione o gestione utente.
- Non definita la politica di aggiornamento o revoca dei token JWT.
- Potenziali complessità di integrazione con eventuali sistemi esterni non specificati.
- Eventuale carenza di risorse con competenze FastAPI rispetto allo stack Java indicato.