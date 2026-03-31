MODULE: PM VERSION: 1
## 1. Obiettivo
Sviluppare un backend per un sistema di autenticazione utenti che consenta la registrazione, login tramite email e password, generazione di token JWT per sessioni autenticati, protezione di endpoint sensibili e una funzionalità base di reset password, utilizzando le tecnologie FastAPI e PostgreSQL.

## 2. Contesto e vincoli
Contesto: Applicazione backend di autenticazione utenti in ambito web o mobile con requisiti di sicurezza base.

Vincoli:
- Deve essere sviluppato con il framework FastAPI.
- Il database utilizzato deve essere PostgreSQL.
- L'autenticazione avviene tramite email e password.
- Il sistema deve generare token JWT per la gestione di sessioni protette.
- La funzione di reset password deve essere implementata in modo base (senza dettagli espliciti, come reset via email o SMS, a meno che non venga successivamente specificato).

## 3. Assunzioni
- La registrazione utente richiede almeno email e password.
- Le password saranno salvate in forma sicura mediante hashing (ad esempio bcrypt).
- Il reset password prevede una procedura semplificata, probabilmente con token temporanei, ma senza dettagli sull'invio email o altre modalità di verifica.
- Il sistema di token JWT includerà almeno un campo identificativo dell’utente e una scadenza.
- Endpoint protetti saranno accessibili solo tramite validazione JWT.
- La gestione utenti (es. disabilitazione account) non è esplicitamente richiesta.

## 4. Scope MVP
- Endpoint API per registrazione utente con validazione dati e salvataggio in PostgreSQL.
- Endpoint login che verifica credenziali e genera token JWT.
- Middleware o dipendenza FastAPI per proteggere gli endpoint tramite verifica JWT.
- Implementazione di almeno un endpoint protetto d’esempio.
- Funzionalità base di reset password (generazione token/reset vulnerabile ma funzionante, senza workflow evoluti).
- Configurazione base di connessione a PostgreSQL e gestione delle migrazioni (se previste).
- Documentazione API minima (es. OpenAPI tramite FastAPI).

## 5. Out of scope
- Implementazioni avanzate di sicurezza (es. MFA).
- Integrazione con sistemi di gestione email per reset password.
- Interfacce utente frontend.
- Audit logging o tracciatura dettagliata degli accessi.
- Gestione avanzata degli utenti (ruoli, permessi, disabilitazioni).
- Supporto a metodi di autenticazione alternativi (OAuth, SSO).
- Deployment e scaling infrastrutturale.

## 6. Task tecnici ordinati
1. Setup ambiente FastAPI con struttura progetto e virtualenv.
2. Configurazione connessione PostgreSQL ed eventuali ORM (SQLAlchemy/others).
3. Definizione schema database per utenti (email, password hashed, reset token, ecc).
4. Implementazione endpoint registrazione utente con validazione e salvataggio.
5. Implementazione endpoint login con verifica password e generazione JWT.
6. Configurazione middleware o dependency FastAPI per verifica token JWT su chiamate protette.
7. Implementazione di esempio endpoint protetto che restituisce risorsa solo se autenticato.
8. Implementazione endpoint reset password base (es. richiesta reset, impostazione nuova password tramite token).
9. Testing unitari e funzionali degli endpoint principali.
10. Documentazione API esposta tramite FastAPI / OpenAPI.
11. Verifica sicurezza base (es. hashing password, scadenza token).

## 7. Acceptance criteria
- Utente può registrarsi fornendo email e password, con verifica di formato e salvataggio nel DB.
- Utente può effettuare login con credenziali corrette e ricevere token JWT valido.
- Endpoint protetti non sono accessibili senza token JWT valido, e permettono l’accesso se il token è corretto.
- Funzionalità reset password permette la generazione di un token reset e la modifica password solo con token valido.
- Tutte le richieste al database sono gestite tramite PostgreSQL con connessione stabile.
- Documentazione API è disponibile e descrive almeno gli endpoint principali (registrazione, login, reset password, endpoint protetto).
- Testing automatici coprono i casi principali di autenticazione e reset password.
- Il sistema gestisce correttamente errori comuni (es. login errato, token scaduto).

## 8. Rischi e punti aperti
- Mancanza di dettagli su modalità di reset password (ad esempio invio email o security question).
- Non specificata la gestione dettagliata della sicurezza JWT (es. algoritmo, secret key, scadenze configurabili).
- Ambiguità riguardo eventuali requisiti di scalabilità o di gestione sessioni multiple.
- Non chiarito se sia necessario audit/logging delle azioni utente.
- La sola dicitura "Reset password (base)" lascia ambigua la definizione esatta del flusso e delle sue protezioni.
- Potenziali complessità nella gestione dei token JWT con revoca e rinnovo, non menzionate esplicitamente.
- Mancanza di dettagli su eventuali regole per la password (es. complessità minima).