MODULE: BA VERSION: 1

## 1. Requisiti funzionali
- Registrazione utente:
  - L'utente deve poter creare un nuovo account fornendo almeno email e password.
  - È implicito che debbano essere validate l'unicità dell'email e la robustezza della password.
- Login con email e password:
  - L'utente deve poter autenticarsi fornendo email e password.
  - Deve essere previsto un controllo di correttezza delle credenziali.
- Generazione token JWT:
  - Il sistema deve generare un JSON Web Token (JWT) al momento del login con credenziali corrette.
  - Il token sarà utilizzato per autenticare le richieste agli endpoint protetti.
- Endpoint protetti:
  - Devono essere presenti endpoint REST API accessibili solo dietro presentazione di un token JWT valido.
- Reset password (base):
  - Deve essere possibile per un utente resettare la propria password.
  - La funzionalità base implica probabilmente invio di un link o codice di reset all'email registrata.
  - Non ci sono dettagli su complicazioni come domande di sicurezza o multifactor authentication.

## 2. Requisiti non funzionali
- Sicurezza:
  - Password devono essere archiviate in forma crittografata (es. hashing sicuro con salt).
  - JWT deve essere firmato e avere una scadenza temporale.
  - Validazione rigorosa degli input per prevenire injection e altre vulnerabilità.
  - Protezione degli endpoint protetti con controllo token e gestione di token scaduti o invalidi.
- Performance:
  - Le API devono rispondere con adeguata velocità per garantire buona esperienza utente.
- Scalabilità:
  - Il sistema backend deve poter scalare orizzontalmente per gestire un numero crescente di utenti e richieste.
- Logging e monitoraggio:
  - Deve essere previsto logging sicuro di eventi critici come tentativi di login/falliti, reset password, errori di sistema.
  - Monitoraggio funzionalità per rilevare malfunzionamenti o attacchi.

## 3. Ambiguità e domande aperte
- Come viene gestita la validazione dell'email in fase di registrazione? Si invia un link di conferma?
  - Assunzione: si prevede almeno una verifica semplice dell’unicità dell’email, ma non è obbligatorio confermare l’email via link.
- Livello di sicurezza richiesto per la password? Minimo caratteri, complessità?
  - Assunzione: applicare best practice standard (es. almeno 8 caratteri, lettere, numeri, simboli).
- Durata e politica di rinnovo del token JWT?
  - Presupponendo token validi per un’ora con possibilità di refresh in futuro.
- Dettagli del reset password? 
  - Assunzione: invio via email di un link o codice temporaneo per poter impostare nuova password.
- Ci sono limiti al numero di tentativi di login o reset per prevenire attacchi brute force?
  - Non specificato, ma si consiglia di prevederli come best practice.

## 4. Edge case e scenari limite
- Tentativi di registrazione con email già utilizzata.
- Login con credenziali errate ripetute numerose volte (rischio di blocco o rate limiting).
- Utilizzo di token JWT scaduti o manomessi per accedere a endpoint protetti.
- Reset password richiesto da un indirizzo IP sospetto o in modo massivo.
- Gestione di utenti che perdono accesso all’email per reset password.
- Concorrenza in registrazioni simultanee con la stessa email.
- Gestione di sessioni multiple per uno stesso utente.

## 5. Dipendenze e vincoli
- La generazione e la verifica dei token JWT è dipendente dalla corretta implementazione del login.
- Gli endpoint protetti dipendono dalla presenza e dalla validità del token JWT.
- Il reset password dipende dalla registrazione di un’email valida per l’invio del link/codice.
- Lo stack tecnologico è vincolato a Java, REST API, Bear come framework, PostgreSQL per il database e JWT per i token.
- L’architettura deve contemplare integrazione con il database PostgreSQL per persistenza dati utente e gestione sessioni o token.

## 6. Requisiti esclusi esplicitamente
- Non è richiesto il supporto per autenticazione multifattore (MFA).
- Non è previsto il login tramite social network o sistemi di terze parti.
- Non sono menzionati ruoli utente o autorizzazioni granulari.
- Non è richiesto un sistema avanzato di gestione delle password (es. password temporanee, scadenze periodiche obbligatorie).
- Non è richiesta una UI o frontend, solo backend.