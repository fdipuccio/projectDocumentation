MODULE: BA VERSION: 1

## 1. Requisiti funzionali
- **Registrazione utente**: 
  - L'utente deve poter creare un account tramite un endpoint dedicato.
  - I dati richiesti per la registrazione includono almeno email e password.
  - Validazione email (formato corretto) e password (criteri minimi di sicurezza, es. lunghezza, complessità) devono essere previste.
  - È implicito che la password venga salvata in modo sicuro, es. hashing.
- **Login con email e password**:
  - L'utente deve poter eseguire il login fornendo email e password.
  - In caso di credenziali errate, deve essere restituito un errore generico senza rivelare dettagli.
- **Generazione token JWT**:
  - Al login corretto, deve essere restituito un token JWT.
  - Il token deve essere firmato e contenere claims appropriati (es. user id, timestamp di scadenza).
  - Si assume un tempo di scadenza configurabile.
- **Endpoint protetti**:
  - Alcuni endpoint del backend devono essere accessibili solo a utenti autenticati tramite validazione del token JWT.
- **Reset password (base)**:
  - Deve essere possibile per un utente richiedere il reset password.
  - La funzionalità base si riferisce presumibilmente ad un processo standard via email con token temporaneo per impostare nuova password.
  - Il sistema deve validare il token di reset e permettere il cambio password in modo sicuro.

## 2. Requisiti non funzionali
- **Sicurezza**: 
  - Tutte le password devono essere gestite con hashing sicuro (es. bcrypt o argon2).
  - I token JWT devono usare algoritmi di firma robusti (es. HS256/RS256).
  - Protezione da attacchi comuni come brute force, SQL injection, e Cross-Site Request Forgery (CSRF) deve essere prevista.
  - Tutti gli input devono essere validati e sanitizzati.
  - Comunicazione tramite HTTPS obbligatoria.
- **Performance**:
  - Il sistema deve garantire tempi di risposta rapidi per la login e registrazione (target da definire).
- **Scalabilità**:
  - Il backend deve poter essere scalato orizzontalmente senza perdita di consistenza nei token JWT o sessioni.
- **Logging e monitoraggio**:
  - Deve essere previsto un sistema di logging degli eventi critici (es. login falliti, reset password).
  - Monitoraggio delle performance e degli errori deve essere implementato.
- **Affidabilità**:
  - Deve essere gestito il ripristino in caso di errore durante la registrazione o reset password (transazioni DB).

## 3. Ambiguità e domande aperte
- **Tecnologia usata**:
  - Il cliente indica FastAPI come vincolo ma lo stack tecnologico riporta Java e REST API, crea ambiguità sull’implementazione. Assumiamo che FastAPI (Python) sia il framework scelto, ma questa discrepanza va chiarita.
- **Dettagli reset password “base”**:
  - Non è specificato se l’invio del link di reset deve avvenire tramite email esterno o sistema interno, e come venga gestito il token di reset (scadenza, crittografia). Assumiamo uso di email con token JWT o UUID sicuro e scadenza breve.
- **Endpoint protetti**:
  - Non è definito quali endpoint e con quali livelli di autorizzazione, o se è previsto un ruolo utente. Assumiamo per ora solo autenticazione di base, senza gestione ruoli.
- **Registrazione dati utente**:
  - Oltre a email e password, ci sono richieste per altri dati obbligatori o opzionali? Assumiamo dati minimi per registrazione.
- **Politiche di blocco account**:
  - È richiesta qualche politica anti-attacco (es. blocco dopo N tentativi falliti)? Non specificato, ma si consiglia.
- **Revoca e rinnovo token JWT**:
  - Non è specificato il meccanismo di revoca token o refresh token. Assumiamo solo token singolo con scadenza.

## 4. Edge case e scenari limite
- Aggiornamento email o password dopo registrazione (non citato ma da considerare come futura esigenza).
- Tentativo di registrazione con email già esistente.
- Reset password richiesto da email non registrata.
- Accesso a endpoint protetti con token scaduto, non valido o mancante.
- Attacchi di replay o token manomessi.
- Gestione simultanea di più richieste di reset password.
- Password deboli o comuni.
- Percorso utente non completato (es. registrazione mai confermata).

## 5. Dipendenze e vincoli
- **Vincoli tecnici**:
  - Utilizzo obbligatorio di FastAPI come framework backend.
  - Database PostgreSQL come sistema di persistenza dati.
- **Dipendenze funzionali**:
  - Login dipende dalla presenza di utente registrato.
  - Generazione token JWT dipende da login avvenuto con successo.
  - Accesso agli endpoint protetti dipende da validità del token JWT.
  - Reset password dipende da validità email utente e gestione token reset.
- **Stack tecnologico discordante**:
  - Stack indicato include Java e REST API, ma requisito vincolare FastAPI (Python) crea dipendenza da chiarire.

## 6. Requisiti esclusi esplicitamente
- Non è richiesto un frontend (solo backend).
- Non è richiesto supporto OAuth o autenticazione tramite terze parti.
- Non è richiesto supporto multi-tenant o gestione ruoli avanzata.
- Non è richiesto un sistema di refresh token o revoca token JWT.
- Non è richiesta la gestione di sessioni o cookie lato client.
- Non è prevista una funzionalità di validazione email tramite link dopo registrazione (non menzionata).