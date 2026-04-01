MODULE: PM VERSION: 1
## 1. Obiettivo
Sviluppare un backend per un sistema di autenticazione utenti che consenta la registrazione, login con email e password, generazione e validazione di token JWT, protezione di endpoint tramite autenticazione e un processo base di reset password. Il sistema deve essere sicuro, scalabile e conforme ai vincoli tecnologici indicati.

## 2. Contesto e vincoli
- Il backend deve essere implementato utilizzando il framework FastAPI (Python).  
- Il database di persistenza dati obbligatorio è PostgreSQL.  
- Deve essere impiegato JWT per l’autenticazione token-based.  
- La comunicazione deve avvenire esclusivamente via HTTPS.  
- Non è previsto front-end né autenticazione tramite terze parti (OAuth).  
- Lo stack attuale indica Java e REST API, ma va chiarito: il vincolo prevalente è FastAPI in Python.  
- Non è richiesta gestione di ruoli utente, multi-tenant, sistemi di refresh o revoca token.  
- Il sistema deve prevedere un logging e monitoraggio degli eventi critici.  
- Sicurezza avanzata con protezioni contro brute force, SQL injection e CSRF.  

## 3. Assunzioni
- Il framework definitivo per il backend sarà FastAPI, non Java; la discrepanza nello stack è un’ambiguità da dirimere prima di sviluppo.  
- Il reset password avviene tramite email esterna con invio di token temporaneo (JWT o UUID sicuro) con breve scadenza.  
- Solo i dati minimi (email e password) sono richiesti per la registrazione, senza ulteriori campi obbligatori.  
- È prevista una politica di blocco account temporaneo dopo un numero configurabile di tentativi falliti (raccomandato ma da confermare).  
- Endpoint protetti sono quelli che necessitano autenticazione base tramite validazione token JWT, senza autorizzazioni o livelli di ruolo.  
- Token JWT sarà unico per sessione, senza meccanismi di refresh o revoca.  
- Password devono essere hashed con algoritmo sicuro (bcrypt o argon2).  
- Il sistema utilizzerà transazioni DB per garantire atomicità nelle operazioni critiche quali registrazione e reset password.  

## 4. Scope MVP
- Endpoint per registrazione utente con validazione email e password, salvataggio hashing password sicuro.  
- Endpoint di login con email e password, restituzione token JWT firmato con claims user id e scadenza configurabile.  
- Middleware o dipendenza FastAPI per validazione token JWT nei singoli endpoint protetti.  
- Endpoint base per richiesta reset password generando token temporaneo inviato via email e chiamata per cambio password con validazione token.  
- Validazioni di input, sanitizzazione dati e comunicazione tramite HTTPS.  
- Sistemi di logging per eventi critici come login falliti, reset password, registrazioni.  
- Gestione errori con risposte generiche per evitare leak di informazioni sensibili.  
- Implementazione difese di base contro attacchi (brute force, injection, CSRF).  
- Utilizzo del database PostgreSQL con transazioni per mantenere consistenza dati.  

## 5. Out of scope
- Frontend o UI di gestione dell’autenticazione.  
- Supporto OAuth o altre autenticazioni esterne.  
- Sistemi di refresh token e revoca token JWT.  
- Gestione multi-tenant o ruoli utente e autorizzazioni granulari.  
- Validazione email tramite link di conferma email post-registrazione.  
- Gestione di sessioni o cookie lato client.  
- Funzionalità avanzate di sicurezza come MFA o CAPTCHA nativa.  

## 6. Task tecnici ordinati
1. Chiarimento e validazione definitiva dello stack tecnologico (FastAPI vs Java).  
2. Definizione schema dati utente minimale (email, password hash e timestamp).  
3. Implementazione endpoint di registrazione con validazione e hashing password (es. bcrypt/argon2).  
4. Implementazione endpoint login con verifica credenziali e generazione token JWT firmato (HS256 o RS256).  
5. Middleware/interceptor per protezione endpoint tramite validazione token JWT.  
6. Progettazione e implementazione endpoint richiesta reset password: generazione token reset temporaneo, invio email esterna.  
7. Endpoint validazione token reset e cambio password sicuro (con hashing).  
8. Implementazione log degli eventi critici su backend.  
9. Applicazione di validazioni input e sanitizzazione per tutti gli endpoint.  
10. Configurazione HTTPS obbligatorio per comunicazione sicura.  
11. Adozione protezione base contro attacchi comuni (es. rate limiting, prevenzione SQL injection/CSRF).  
12. Test di scenario limitazioni: email duplicate, token scaduti/non validi, tentativi multipli reset.  
13. Gestione transazioni DB per garantire rollback in errori durante registrazione o reset password.  
14. Documentazione API e criteri di accettazione.  

## 7. Acceptance criteria
- Registrazione crea un account solo con email valida e password che rispettano criteri sicurezza, con password salvata hashed.  
- Login con credenziali corrette ritorna token JWT firmato contenente user id e scadenza configurabile.  
- Login con credenziali non valide restituisce errore generico senza informazioni di dettaglio.  
- Endpoint protetti sono accessibili solo con token JWT valido e attivo, altrimenti restituiscono errore 401.  
- Reset password: richiesta da email registrata genera invio token temporaneo via email; cambio password solo se token valido e non scaduto.  
- Tutti input sono validati e sanitizzati; nessuna vulnerabilità nota (SQL injection, CSRF) presente.  
- Logging registrato per eventi critici come login fallito, reset password richiesti.  
- Comunicazione esclusivamente su protocollo HTTPS.  
- Funzionalità gestita con transazioni DB per garantire consistenza dati e rollback in caso di errori.  
- Sistema dimostra mitigazione basilare contro brute force (es. blocco o rate limiting configuroabile).  

## 8. Rischi e punti aperti
- Ambiguità significativa riguardo tecnologia backend (FastAPI richiesto vs stack originale Java), da risolvere prima di sviluppo.  
- Mancanza di dettaglio su gestione token reset (tipo token JWT o UUID, crittografia, scadenza precisa).  
- Assenza di definizione precisa delle politiche di sicurezza nei blocchi account (tentativi falliti, tempistiche).  
- Mancata definizione esatta degli endpoint protetti e loro estensione futura (oggi solo autenticazione, non autorizzazione dettagliata).  
- Assenza di meccanismi di revoca o refresh token JWT potrebbe limitare flessibilità e sicurezza.  
- Necessità di definire target performance per tempi di risposta (indicato ma non quantificato dal BA).  
- Gestione del concurrency su reset password e registrazione (es. richieste multiple simultanee) da testare accuratamente.  
- Mancanza di funzionalità di validazione email post-registrazione potrebbe causare casi di email inesistenti accettate.  
- Rischio che l’ambiguità tecnologica impatti la calendarizzazione e la composizione del team di sviluppo.