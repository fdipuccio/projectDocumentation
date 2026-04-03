MODULE: PM VERSION: 1
## 1. Obiettivo
Sviluppare un backend REST API per un sistema di autenticazione utenti che consenta la registrazione, login, gestione token JWT, protezione degli endpoint e reset password di base, garantendo sicurezza, performance e conformità GDPR.

## 2. Contesto e vincoli
- Tecnologia: Java, REST API, Bear, PostgreSQL, JWT.
- Lato frontend non previsto, solo backend.
- Comunicazioni API su HTTPS obbligatorie.
- Database PostgreSQL usato per gestire unicità email e persistenza dati.
- Token JWT necessari per proteggere gli endpoint eccetto registrazione e login.
- Nessun meccanismo di refresh token previsto.
- Nessuna conferma email post-registrazione.
- Limitazioni su tentativi di login e reset password per mitigare attacchi brute force.
- Conformità GDPR nella gestione dei dati utente.
- Logging strutturato e monitoraggio di eventi critici.
- Performance target: risposta in meno di 500 ms per login e registrazione.
- È necessario gestire scenari con perdita temporanea del database.

## 3. Assunzioni
- Durata token JWT configurabile tra 15 e 60 minuti, default 30 minuti.
- Payload JWT minimo con identificativo utente, ulteriori claims possibili in futuro.
- Limite tentativi login e reset password: 5 tentativi con blocco di 15 minuti.
- Logging eventi include login, errori, reset password e accessi protetti.
- Nessuna attivazione account tramite conferma email.
- Il sistema non gestisce ruoli utente o autorizzazioni complesse.

## 4. Scope MVP
- Registrazione utente con validazione email e password (minimo 8 caratteri con lettere e numeri).
- Login con email e password, gestione errori credenziali e limitazione tentativi.
- Generazione e validazione token JWT sicuri con durata configurabile.
- Protezione di tutti gli endpoint tranne registrazione e login tramite validazione token JWT.
- Flusso base di reset password con invio token via email temporaneo a scadenza.
- Gestione errori per token JWT scaduto/invalido e reset password scaduto o già usato.
- Salvataggio password con hashing e salting tramite bcrypt.
- Logging strutturato degli eventi di autenticazione e sicurezza.
- Gestione errori per input non validi.
- Monitoraggio tentativi sospetti.

## 5. Out of scope
- Conferma email post-registrazione.
- Meccanismi di refresh token JWT.
- Implementazione frontend o interfacce utente.
- Gestione ruoli utente e autorizzazioni avanzate.
- Integrazione con provider esterni di autenticazione (OAuth, SSO).

## 6. Task tecnici ordinati
1. Progettazione schema database PostgreSQL per utenti (email unica, password hashata).
2. Implementazione endpoint REST per registrazione utente con validazione input.
3. Implementazione endpoint login con gestione errori e limitazione tentativi.
4. Sviluppo meccanismo generazione token JWT con payload minimo e durata configurabile.
5. Middleware per validazione token JWT su endpoint protetti.
6. Implementazione endpoint reset password: generazione, invio token via email e modifica password.
7. Gestione controllo e blocco tentativi login/reset password (soglia e tempistica).
8. Implementazione hashing e salting password con bcrypt.
9. Logging strutturato di eventi critici (login, errori, reset, accessi).
10. Monitoraggio sicurezza e tentativi sospetti.
11. Gestione errori e scenari limite (input non valido, token scaduto, email già presente).
12. Test di performance per rispettare il limite < 500 ms.
13. Gestione resilienza e riavvio in caso di perdita temporanea del database.

## 7. Acceptance criteria
- Utenti possono registrarsi solo con email unica validata e password conforme.
- Login fallisce dopo 5 tentativi errati con blocco di 15 minuti.
- Token JWT generati con durata di default 30 minuti e payload contenente userId.
- Tutti gli endpoint eccetto registrazione e login richiedono token JWT valido; accesso negato per token assente, scaduto o invalido.
- Password salvate con bcrypt hashing e salting.
- Reset password invia token via email, token scade dopo 15-60 minuti e non può essere riutilizzato.
- Logging conforme a specifica coprendo eventi di autenticazione, errori e accessi.
- Performance login e registrazione sotto 500 ms.
- Sistema gestisce correttamente situazioni di perdita temporanea del database e scenari limite.
- Il sistema è conforme GDPR nella gestione dei dati utente e comunica esclusivamente via HTTPS.

## 8. Rischi e punti aperti
- Definire esattamente la soglia di tentativi e durata del blocco per login e reset password.
- Dettagliare formato e livello di logging necessario per monitoraggio efficace.
- Gestione email funzionante e affidabile per invio token reset password.
- Eventuale estensione payload JWT e claims da definire in futuro.
- Possibili problemi di performance o scalabilità sotto carichi elevati da monitorare.
- Resilienza del sistema in caso di perdita temporanea o degrado del database da implementare e testare.
- Assenza di meccanismi di refresh token potrebbe limitare usabilità token lunghi.
- Mancanza di conferma email potrebbe comportare rischi di account non verificati (da valutare nell'ambito sicurezza).