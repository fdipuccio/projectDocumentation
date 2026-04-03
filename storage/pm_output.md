MODULE: PM VERSION: 1
## 1. Obiettivo
Realizzare un backend di autenticazione utenti che permetta la registrazione, login tramite email e password, generazione e verifica di token JWT per l’accesso a endpoint protetti, e reset password base, sfruttando lo stack tecnologico Java, REST API, Bear framework, PostgreSQL e JWT. Il sistema deve essere sicuro, performante, scalabile e facile da monitorare.

## 2. Contesto e vincoli
- Stack tecnico obbligatorio: Java, REST API, Bear framework, PostgreSQL come database, JWT per i token di autenticazione.
- Backend only: non è prevista una UI o frontend.
- L’architettura deve prevedere integrazione completa con il database PostgreSQL per persistenza dati utenti e gestione sessioni/token.
- La sicurezza prevede archiviazione password hashed con salt, JWT firmati con scadenza temporale, e controlli rigorosi sugli input.
- Le API devono rispondere con buona velocità per garantire esperienza utente soddisfacente.
- Il sistema deve poter scalare orizzontalmente.
- Logging sicuro e monitoraggio di eventi critici sono obbligatori.
- Non sono previsti sistemi di autenticazione multifattoriale, social login o autorizzazioni granulari.

## 3. Assunzioni
- La validazione email in fase di registrazione includerà verifica dell’unicità ma non necessariamente conferma tramite link di attivazione.
- Le password devono rispettare best practice standard: minimo 8 caratteri con lettere, numeri e simboli.
- I token JWT avranno durata di circa un’ora con possibilità di introdurre in futuro meccanismi di refresh.
- Il reset password base si realizzerà inviando un link o codice temporaneo via email.
- Si introdurranno limiti di tentativi login/reset per mitigare attacchi brute force, pur non essendo esplicitamente richiesto.
- Non si implementano sistemi avanzati di gestione password come scadenze obbligatorie o password temporanee.
- La gestione di sessioni multiple per utente sarà supportata senza restrizioni specifiche.

## 4. Scope MVP
- Endpoint REST per registrazione utente con controllo unicità email e validazione password.
- Endpoint login con verifica credenziali e generazione token JWT firmato e con scadenza.
- Middleware o filtro per protezione endpoint accessibili solo con token JWT valido.
- Endpoint per reset password base, con invio codice/link a email registrata.
- Persistenza dati utenti e token/sessioni su PostgreSQL.
- Logging sicuro e monitoraggio degli eventi critici (login, reset password, errori).
- Gestione degli errori e risposte coerenti per casi di credenziali errate, token invalidi/scaduti.
- Meccanismi base di sicurezza input validation per prevenire injection e vulnerabilità comuni.
- Scalabilità a livello di backend e database.

## 5. Out of scope
- Autenticazione multifattore (MFA).
- Login tramite social network o sistemi terzi.
- Gestione ruoli utente e autorizzazioni granulari.
- UI o frontend.
- Gestione avanzata di password (es. password temporanee, politiche di scadenza obbligatorie).
- Sistemi complessi di verifica identità o blocco IP automatico oltre limiti di tentativi login/reset (non specificati esplicitamente).

## 6. Task tecnici ordinati
1. Progettazione schema database utenti e token in PostgreSQL.
2. Implementazione endpoint registrazione con validazione input, controllo unicità email, hashing password con salt sicuro.
3. Implementazione endpoint login con verifica credenziali e generazione token JWT firmato, impostazione scadenza token.
4. Sviluppo middleware Bear per protezione endpoint mediante verifica token JWT valido.
5. Implementazione endpoint reset password base con generazione codice/link temporaneo e invio email.
6. Configurazione logging sicuro per eventi critici (login, reset, errori).
7. Realizzazione controlli input rigorosi per tutte le API.
8. Testing funzionale e di sicurezza su casi normali e limitrofi (tentativi errati, token scaduti).
9. Setup monitoraggio e alert base per malfunzionamenti e tentativi di attacco.
10. Documentazione tecnica API e configurazioni sicurezza.

## 7. Acceptance criteria
- Utente può registrarsi con email unica e password conforme a policy.
- Utente può effettuare login con email/password valide e ricevere token JWT con scadenza.
- Endpoint protetti rispondono solo se token JWT è presente, valido e non scaduto; altrimenti ritornano errore 401.
- Reset password consente di inviare codice/link via email e permette modifica password.
- Password salvate nel database sono hashed con salt sicuro.
- Tutte le API validano gli input e impediscono injection o dati non validi.
- Log eventi critici sono correttamente registrati e consultabili in modo sicuro.
- Il sistema risponde con performance accettabili (es. tempi di risposta inferiori a 1 secondo per endpoint critici).
- Il sistema scala orizzontalmente senza perdita di funzionalità.
- Test di sicurezza base superati (es. tentativi ripetuti login bloccati o limitati).
- Documentazione tecnica completa e aggiornata.

## 8. Rischi e punti aperti
- Ambiguità sulla conferma dell’email in fase di registrazione, attualmente esclusa ma possibile rischio di account non validi.
- Politiche di sicurezza password e limiti di tentativi non dettagliati: serve conferma o definizione più precisa.
- Gestione token JWT: politiche di rinnovo/refresh non implementate nell’MVP possono limitare usabilità futura.
- Vulnerabilità potenziali a DoS o attacchi brute force se limiti tentativi o monitoraggio non adeguati.
- Gestione utenti senza accesso all’email per reset password non risolta.
- Concorrenza in registrazione simultanea con stessa email deve essere gestita attentamente lato DB.
- Necessità di chiarire dettagli di logging e monitoraggio per rispettare eventuali compliance di sicurezza.
- Mancanza di ruoli o autorizzazioni granulari potrebbe limitare l’estensione futura del sistema.