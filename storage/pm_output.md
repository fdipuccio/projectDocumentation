MODULE: PM VERSION: 1
## 1. Obiettivo
Sviluppare un backend per un sistema di autenticazione utenti che consenta la registrazione, il login tramite email e password, la generazione e gestione di token JWT per l’accesso controllato a endpoint protetti, e una funzionalità base di reset della password.

## 2. Contesto e vincoli
- Il progetto deve essere implementato utilizzando il framework FastAPI.
- Il database di persistenza dovrà essere PostgreSQL.
- Lo stack tecnico è Python con librerie per JWT.
- I token JWT devono essere generati e validati seguendo standard sicuri.
- Il sistema deve prevedere endpoint REST accessibili solo previa autenticazione tramite JWT.
- La funzionalità di reset password deve essere di base (non dettagliata) e implementata solo lato backend.
- Non sono menzionate specifiche di infrastruttura, sicurezza avanzata (es. MFA) o interfacce utente.

## 3. Assunzioni
- L’email utente è unica e costituisce l’identificativo principale per login e registrazione.
- La gestione delle email per reset password (es. invio email) non è parte del progetto o sarà gestita esternamente.
- Il JWT conterrà almeno il minimo necessario per identificare l’utente e la scadenza del token.
- Non sono richiesti livelli di sicurezza superiori (es. rate limiting, captcha).
- Il backend esporrà API REST, con interfacce documentabili (es. OpenAPI).
- La password sarà salvata in forma sicura (hashata) ma la specifica tecnica di hashing non è stata fornita.
- La scalabilità e la gestione dell’infrastruttura non sono considerate MVP.

## 4. Scope MVP
- Endpoint per registrazione utente con validazione base dati e salvataggio su PostgreSQL.
- Endpoint per login con verifica credenziali, generazione e restituzione token JWT.
- Middleware o sistema di protezione endpoint che richieda token JWT valido.
- Endpoint protetti accessibili solo tramite token JWT.
- Endpoint per reset password con logica base (es. richiesta reset e aggiornamento password).
- Persistenza dati utenti e token necessari su PostgreSQL.

## 5. Out of scope
- Interfaccia front-end o mobile.
- Sistema di invio email o notifiche per reset password.
- Autenticazione social o multi-factor.
- Gestione dettagliata del ciclo di vita del token oltre la generazione e validazione di base.
- Funzionalità avanzate di sicurezza (logging dettagliato degli accessi, monitoraggio intrusioni).
- Documentazione esterna non tecnica o supporto post-release.
- Scalabilità orizzontale, deployment, e configurazioni infrastrutturali.

## 6. Task tecnici ordinati
1. Definizione schema database utenti e eventuali tabelle per gestione reset password.
2. Implementazione modello dati in PostgreSQL.
3. Setup progetto FastAPI con configurazione database.
4. Sviluppo endpoint registrazione utente:
   - Validazione dati input
   - Hash password e salvataggio
5. Sviluppo endpoint login:
   - Verifica credenziali
   - Generazione token JWT con payload minimo e scadenza
6. Implementazione middleware o dipendenza FastAPI per protezione endpoint tramite verifica token JWT.
7. Sviluppo endpoint protetti di prova (es. /protected) che richiedano JWT valido.
8. Implementazione reset password base:
   - Endpoint richiesta reset (es. generazione token reset o flag)
   - Endpoint aggiornamento nuova password
9. Testing unitario e integrazione per endpoint e logica token.
10. Documentazione API (OpenAPI/Swagger automatica con FastAPI).

## 7. Acceptance criteria
- Utente può registrarsi con email e password e i dati sono correttamente salvati e protetti nel DB.
- Utente può autenticarsi con email e password e ricevere un token JWT valido.
- Token JWT è verificato e utilizzato per l’accesso a endpoint protetti, mentre richieste senza token o con token scaduto/errato ricevono errore 401.
- Endpoint reset password consente la richiesta e l’aggiornamento della password utente, con validazione base.
- Il backend è sviluppato usando FastAPI e PostgreSQL come DB.
- I test automatizzati coprono almeno i casi critici di registrazione, login, protezione endpoint e reset password.
- La documentazione API è fruibile tramite OpenAPI/Swagger.

## 8. Rischi e punti aperti
- Non è specificato il metodo di comunicazione o validazione per il reset password (es. email token), che potrebbe essere critico per sicurezza e usabilità.
- Nessuna indicazione sulla politica di sicurezza password (lunghezza minima, complessità).
- Gestione scadenza e revoca token JWT non dettagliata, possibile rischio sicurezza a lungo termine.
- Non definito come gestire email duplicate o utenti già esistenti.
- Non specificata la versione minima di FastAPI, PostgreSQL o librerie Python JWT.
- Mancanza dell’indicazione su eventuali requisiti GDPR o compliance privacy nel trattamento dati utente.