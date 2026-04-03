MODULE: BA VERSION: 1
## 1. Requisiti funzionali
- Registrazione utente: Il sistema deve permettere la registrazione di un nuovo utente con i dati necessari (es. email, password). È implicito che la password debba essere validata e memorizzata in modo sicuro (hashing). Non è specificata la necessità di conferma email o altri dati aggiuntivi.
- Login con email e password: Il sistema deve consentire agli utenti di autenticarsi usando email e password. Deve essere prevista la validazione delle credenziali e la gestione degli errori in caso di login fallito.
- Generazione token JWT: Al login avvenuto con successo, il sistema deve generare un token JWT per l’utente, utilizzabile per autenticare le richieste successive.
- Endpoint protetti: Alcuni endpoint delle API devono essere protetti e accessibili solo a utenti autenticati tramite token JWT valido.
- Reset password (base): Deve essere previsto un endpoint per il reset password, considerato "base", quindi presumibilmente semplice (es. invio link o codice di reset via email). Non sono specificati flussi avanzati come domande di sicurezza o autenticazione a due fattori.

## 2. Requisiti non funzionali
- Sicurezza: Le password devono essere memorizzate in forma hashed e salted. I token JWT devono essere firmati con chiave sicura. Ogni input deve essere validato per prevenire injection o attacchi correlati. Le comunicazioni devono essere protette (HTTPS).
- Performance: Le operazioni di autenticazione devono essere efficienti per non introdurre latenza significativa. Il database deve essere indicizzato per ricerche veloci su email utente.
- Scalabilità: Il backend deve poter scalare orizzontalmente senza compromettere la validità dei token JWT (stateless).
- Logging e monitoraggio: Deve essere previsto logging centralizzato degli eventi di autenticazione (login, reset password, errori), mantenendo la privacy degli utenti. Monitoraggio delle performance e sicurezza delle API.
- Compatibilità con stack: Deve usare Java per il backend, REST API come protocollo di comunicazione, Bear framework per lo sviluppo, PostgreSQL come database e JWT come metodo di autenticazione.

## 3. Ambiguità e domande aperte
- Quali dati sono obbligatori per la registrazione utente oltre a email e password? Assunzione: solo email e password.
- È richiesta la verifica della validità dell’email via conferma? Non specificato, assumiamo che non sia richiesta in questa versione.
- Il reset password "base" prevede l’invio via email di un link o codice di reset? Assumiamo il metodo via email con link temporaneo.
- La durata di validità del token JWT e la gestione della revoca token non sono specificate. Assumiamo un token con scadenza predefinita (es. 1 ora) e nessuna revoca attiva.
- Sono previsti livelli di accesso o ruoli utenti? Non menzionato, quindi il sistema sarà a singolo livello di utenza.
- Quali sono gli endpoint da proteggere? Assumiamo che tutti gli endpoint tranne registrazione, login e reset password siano protetti.

## 4. Edge case e scenari limite
- Tentativi multipli di login falliti: gestione di blocco temporaneo o meccanismo anti-brute force non specificato.
- Reset password richiesto da un'email non registrata: come trattare questa situazione? Assunzione: rispondere con messaggio generico per sicurezza.
- Token JWT scaduto o manomesso: come gestire la risposta e la richiesta di nuovo login.
- Cosa succede se un utente prova a registrarsi con email già esistente? Gestione errore duplicati.
- Password troppo debole o non conforme a criteri di sicurezza.
- Problemi di sincronizzazione del tempo per expiry token JWT in scenari di sistemi distribuiti.

## 5. Dipendenze e vincoli
- La generazione del token JWT dipende dal login con successo tramite email e password.
- L’accesso agli endpoint protetti dipende dalla validità del token JWT.
- Il reset password dipende dalla possibilità di identificare un utente tramite email e inviare comunicazioni esterne (email).
- Stack tecnologico vincola l’implementazione a Java con REST API, uso del framework Bear, il database PostgreSQL e utilizzo di token JWT per gestione sessione.
- La sicurezza dipende dalla corretta configurazione del sistema di hashing password, firma JWT e uso di HTTPS.

## 6. Requisiti esclusi esplicitamente
- Non è richiesto il supporto a autenticazione tramite social login o OAuth.
- Non è richiesta la gestione di ruoli o permessi differenziati per utenti.
- Non è prevista la conferma via email per la registrazione.
- Non sono previsti meccanismi avanzati di reset password oltre a una funzionalità base (es. nessuna autenticazione a due fattori).
- Non è indicata la necessità di interfacce di amministrazione o gestione utenti.
- Non è menzionata la localizzazione o supporto multilingua.