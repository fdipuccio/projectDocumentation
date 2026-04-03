MODULE: BA VERSION: 1
## 1. Requisiti funzionali
- Registrazione utente: Gli utenti devono poter registrarsi fornendo email e password. È richiesta la validazione del formato email, verifica unicità email nel sistema, e il rispetto di requisiti minimi di complessità della password (minimo 8 caratteri con combinazione di lettere e numeri). Non è prevista una conferma via email per l'attivazione dell'account.
- Login con email e password: Gli utenti devono poter effettuare login con email e password registrate, con gestione degli errori di credenziali e con limitazione dei tentativi errati per prevenire attacchi brute force.
- Generazione token JWT: Dopo login, deve essere generato un token JWT firmato con chiavi sicure, dotato di payload contenente almeno l'identificativo utente e con durata configurabile tra 15 e 60 minuti. Non è previsto un meccanismo di refresh token.
- Endpoint protetti: Tutti gli endpoint, ad eccezione di quelli di registrazione e login, devono essere protetti richiedendo la validazione del token JWT nel header Authorization. Accessi con token mancante, scaduto o invalido devono essere gestiti con risposta di errore.
- Reset password (base): Deve essere previsto un flusso base di reset password che prevede l'invio di un token temporaneo, univoco e a scadenza (15-60 minuti) via email, che consenta la modifica della password. Gestione degli scenari con token scaduto o utilizzato.

## 2. Requisiti non funzionali
- Sicurezza: Password memorizzate con hashing e salting usando bcrypt. Tutte le comunicazioni API devono avvenire su HTTPS. Validazione e sanitizzazione degli input per prevenire attacchi di injection. Limitazione dei tentativi per login e reset password per evitare brute force. Il sistema deve essere conforme GDPR nella gestione dati utente.
- Performance: Tempo di risposta per login e registrazione inferiore a 500 ms.
- Scalabilità: Gestione efficiente del database PostgreSQL per supportare unicità email e operazioni di autenticazione. Sistema resiliente alla perdita temporanea del DB.
- Logging e monitoraggio: Logging strutturato di eventi di autenticazione, accessi, errori e operazioni critiche. Monitoraggio di tentativi sospetti di login e altri eventi di sicurezza.

## 3. Ambiguità e domande aperte
- Conferma email post-registrazione non esplicitamente richiesta: Assumiamo che non sia prevista attivazione account con conferma email.
- Durata esatta token JWT: Propone configurazione da 15 a 60 minuti, si consiglia definire un valore di default (es. 30 minuti).
- Dettagli payload JWT e claims: Assumiamo payload minimo con userId, eventuali estensioni da definire successivamente.
- Limite tentativi login e reset password: Numero esatto di tentativi e durata del blocco non specificati, si propone di definire una soglia ragionevole (es. 5 tentativi con blocco temporaneo di 15 minuti).
- Logging: Dettaglio degli eventi loggati e formato non specificati, si assume logging di eventi login, errori, reset password e accessi protetti.

## 4. Edge case e scenari limite
- Tentativi di registrazione con email già presente: il sistema deve gestire con errore specifico.
- Login con credenziali errate multiple volte: blocco o delay progressivo per mitigare brute force.
- Accesso a endpoint protetti con token scaduto o invalido.
- Token reset password scaduto o già usato.
- Possibile perdita temporanea del database o problemi di accesso.
- Validazione input non conforme (email errata, password troppo debole).

## 5. Dipendenze e vincoli
- Dipendenza critica tra registrazione e unicità email nel DB PostgreSQL.
- I token JWT sono fondamentali per l'accesso a endpoint protetti.
- Reset password presuppone l'invio email funzionante e gestione sicura dei token temporanei.
- Sicurezza complessiva dipende da corretta gestione di password, token e comunicazioni HTTPS.
- Limitazione tentativi login e reset necessita sincronizzazione con logging per monitoraggio.

## 6. Requisiti esclusi esplicitamente
- Non è richiesta conferma email post-registrazione (attivazione account).
- Non è previsto meccanismo di refresh token JWT.
- Non sono richiesti ruoli utente o livelli di autorizzazione complessi.
- Non è prevista integrazione con provider esterni di autenticazione (OAuth, SSO).
- Non è previsto frontend o interfacce utente, solo backend REST API.