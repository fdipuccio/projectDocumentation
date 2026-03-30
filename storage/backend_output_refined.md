MODULE: BACKEND
VERSION: 1

## 1. Obiettivo backend
Realizzare un modulo backend RESTful API, conforme ai requisiti core del progetto PM MVP, che garantisca la fruizione corretta e sicura delle funzionalità richieste. Il backend deve essere integrato con sistemi legacy tramite API standard, conforme alle policy di sicurezza aziendali (inclusa gestione JWT per autenticazione/autorizzazione), e utilizzare PostgreSQL per la persistenza dati con un’architettura modulare e manutenibile, predisposta per future estensioni.

## 2. Assunzioni tecniche
- Lo stack tecnologico è Python con FastAPI come framework backend, PostgreSQL per DB relazionale, e JWT per gestione sicurezza.
- Infrastruttura runtime, CI/CD e ambiente di test già predisposti.
- Sistemi legacy esposti tramite API REST standard utilizzate per integrazione.
- Requisiti MVP già definiti, non sono previste funzionalità avanzate o ottimizzazioni di performance nella prima release.
- Disponibilità di competenze interne per collaborazione su sicurezza, infrastruttura e test.
- Gestione sicura delle chiavi JWT e segreti sarà formalizzata e integrata con la configurazione di ambiente.
- Requisiti di performance e security test saranno definiti e formalizzati in collaborazione con team dedicati.

## 3. Architettura backend
Architettura a 3 livelli:
- **Presentation Layer**: API FastAPI esposte all’esterno, gestiscono richieste HTTP e orchestrano la business logic.
- **Business Logic Layer**: servizi modulari e testabili che implementano le regole di business del modulo PM.
- **Data Access Layer**: utilizzo di ORM (es. SQLAlchemy o equivalent ORM) per l’accesso sicuro e manutenibile a PostgreSQL.
L’architettura prevede componenti per integrazione con sistemi esterni tramite client HTTP con gestione retry leggera e circuit breaker soft configurabili. La gestione degli errori e logging sono centralizzati. Autenticazione e autorizzazione con JWT sono integrate a livello API gateway o middleware FastAPI.

## 4. Moduli e responsabilità
- **API Module**: definizione degli endpoint REST, validazione input/output, gestione sicurezza JWT.
- **Service Layer**: implementazione della logica applicativa core, orchestrazione chiamate a repository e servizi esterni.
- **Repository Layer**: interfaccia di accesso a dati persistenti, con astrazione ORM su PostgreSQL.
- **Integration Module**: client per comunicazione con sistemi legacy/API esterne, inclusa gestione retry e circuit breaker semplificati.
- **Security Module**: utility per validazione token JWT, gestione permessi.
- **Error Handling Module**: centralizzazione errori, mapping exception → response API standard.
- **Logging Module**: gestione logging strutturato conforme a standard enterprise.
- **Config Module**: gestione configurazioni ambiente incluse variabili di segretezza (JWT secret management).

## 5. API principali
- **POST /auth/login** — autenticazione utente, generazione token JWT.
- **GET /resource/{id}** — recupero dati core del modulo PM (esempio generico).
- **POST /resource** — creazione di una nuova risorsa gestita dal modulo.
- **PUT /resource/{id}** — aggiornamento di risorsa.
- **DELETE /resource/{id}** — rimozione di risorsa (se prevista dal requisito).
- Endpoint supplementari per gestione configurazioni o monitoraggio modulo, se richiesti.
Tutti gli endpoint sono protetti con JWT e prevedono controlli di autorizzazione di base.

## 6. Business logic
- Validazione e trasformazione dati in ingresso e uscita API.
- Coordinamento delle operazioni persistenti su PostgreSQL tramite repository.
- Gestione regole di accesso e permessi utente integrate con JWT.
- Coordinamento con sistemi legacy tramite modulo integrazione, includendo gestione retry normali su errori transitori.
- Applicazione delle policy di sicurezza e governance interne.
- Gestione coerenza dati minima a livello di transazioni DB.
- Preparazione della risposta API comprensibile e conforme a standard REST.

## 7. Persistenza e integrazioni
- Persistenza dati tramite PostgreSQL, accesso mediato da ORM per astrarre query e garantire sicurezza.
- Mappatura chiara tra modelli di dominio e tabelle DB per mantenibilità.
- Integrazione con sistemi legacy via API HTTP REST, con client che implementano retry policy basilare e circuit breaker soft.
- Gestione configurazioni dinamiche di endpoint legacy, timeout, e policy retry nella configurazione ambiente.
- Log delle chiamate esterne ed esiti per audit e troubleshooting.

## 8. Autenticazione e autorizzazione
- Autenticazione basata su JWT: endpoint login genera token firmati.
- Middleware FastAPI valida token JWT su ogni richiesta protetta.
- Token contengono claims per definizione ruoli e permessi.
- Gestione scadenza e rinnovo token conforme policy.
- Segreti JWT (chiavi di firma/verifica) gestiti in modo sicuro attraverso variabili di ambiente o vault.
- Logging e monitoraggio accessi e errori di autenticazione.

## 9. Gestione errori
- Centralizzazione gestione eccezioni con middleware FastAPI custom.
- Risposta API uniforme con codici HTTP chiari (4xx per errori client, 5xx per errori server).
- Logging strutturato di errori con dettagli contestuali per debugging.
- Gestione errori di integrazione legacy (timeout, errori HTTP) con log e propagazione controllata.
- Eventuale fallback semplice o messaggi di errore user-friendly per chiamate API esterne non disponibili.
- Monitoraggio errori per alerting (infrastruttura esterna non prevista dal scope).

## 10. Strategia di test backend
- Test unitari per tutti i moduli di business logic e repository con copertura >80%.
- Test di integrazione per endpoint FastAPI e interfaccia con DB PostgreSQL.
- Test di integrazione simulando chiamate a sistemi legacy con scenari di successo ed errore.
- Test funzionali end-to-end in ambiente di test pre-rilascio.
- Test di sicurezza base (JWT validation, access control) integrati nella pipeline CI/CD.
- Definizione e pianificazione di test di penetrazione, stress test e audit di sicurezza in collaborazione con team infrastructure/security.
- Review continua post rilasci MVP per estendere piano test.

## 11. Rischi tecnici
- Requisiti parziali o ambigui che possono generare estensioni non previste, rallentando rilascio MVP.
- Integrazione con sistemi legacy imprevedibili o non documentati che possono generare malfunzionamenti.
- Mancanza di definizione chiara e formalizzazione test performance e sicurezza potrebbe compromettere qualità rilascio.
- Possibili criticità nella gestione sicura dei segreti JWT in ambiente produzione se non coordinato con team infrastructure.
- Dipendenze da altri team o fornitori esterni per API o infrastruttura senza accordi formali.
- Limitata esperienza o tools per monitoraggio runtime e resilienza nel MVP, possibile impatto su stabilità.

## 12. Struttura file proposta
- `/app`
  - `/api`
    - `routes.py` — definizione endpoint FastAPI.
    - `schemas.py` — serializzazione/validazione dati input/output con Pydantic.
  - `/services`
    - `core.py` — implementazione logica business principale.
    - `integration.py` — client API verso sistemi legacy.
    - `security.py` — gestione JWT, autenticazione e autorizzazione.
  - `/repositories`
    - `models.py` — modelli ORM per PostgreSQL.
    - `db.py` — sessioni DB e operazioni CRUD.
  - `/config`
    - `settings.py` — configurazione ambiente e parametri runtime.
  - `/exceptions`
    - `handlers.py` — gestione e mapping errori API.
  - `/logging`
    - `logger.py` — setup logging strutturato.
- `/tests`
  - `/unit` — test unitari per moduli.
  - `/integration` — test integrazione API e DB.
  - `/e2e` — test end-to-end.
- `main.py` — entrypoint FastAPI app.

## 13. Piano di implementazione
1. **Analisi e definizione dettagliata stack e policy di sicurezza**: collaborare con team infrastruttura per formalizzare gestione segreti JWT, configurazioni sicure e criteri test specifici di sicurezza e performance.
2. **Progettazione dettagliata architettura e definizione API**: stabilire tutti gli endpoint REST necessari per MVP, struttura moduli e pattern di integrazione con sistemi legacy.
3. **Sviluppo moduli core e integrazioni**: implementare logica business, meccanismi di persistenza, middleware di autenticazione e client integrazione, supportando gestione errori centralizzata.
4. **Test, validazione e documentazione tecnica**: scrittura test unitari e di integrazione, esecuzione test funzionali, redazione documentazione API e operativa, setup pipeline CI/CD.
5. **Review con Product Owner e stakeholder**: verifica completezza funzionalità, compliance policy e readiness rilascio ambiente concordato senza regressioni.
6. **Rilascio MVP e monitoraggio iniziale**: deploy ambiente di produzione con monitoraggio errori e log di sistema, pianificazione iterazioni successive per estensioni e ottimizzazioni.