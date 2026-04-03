MODULE: PM VERSION: 1
## 1. Obiettivo
Sviluppare un backend per un sistema di autenticazione utenti che consenta la registrazione, login tramite email e password, generazione di token JWT per sessioni stateless, protezione degli endpoint API e reset password base, in conformità allo stack tecnologico definito.

## 2. Contesto e vincoli
- Stack tecnologico vincolato a Java per il backend, REST API per l'interfaccia, framework Bear per lo sviluppo, PostgreSQL come database e JWT per la gestione dell'autenticazione.
- Sicurezza obbligatoria: password memorizzate in forma hashed e salted, token JWT firmati con chiave sicura, comunicazioni protette via HTTPS e validazione input per prevenzione injection.
- Architettura stateless per consentire scalabilità orizzontale senza compromettere la validità dei token JWT.
- Logging centralizzato obbligatorio su eventi di autenticazione mantenendo la privacy utenti.
- Il sistema deve escludere esplicitamente autenticazione tramite social login, gestione ruoli, conferma email, meccanismi avanzati di reset password e interfacce di amministrazione.

## 3. Assunzioni
- I dati obbligatori per la registrazione sono solo email e password.
- Non è prevista conferma o validazione espressa dell'email tramite link o codice.
- Il reset password base prevede l’invio di link temporaneo via email.
- Token JWT ha una scadenza fissa (es. 1 ora), senza meccanismi di revoca attivi.
- Tutti gli endpoint sono protetti da autenticazione JWT tranne quelli di registrazione, login e reset password.
- Gestione degli errori con messaggi generici per sicurezza (es. reset a email non registrata).
- Assenza di meccanismi anti-brute force o blocchi temporanei per login falliti a meno che non vengano esplicitamente richiesti successivamente.

## 4. Scope MVP
- Endpoint di registrazione utente con validazione base e gestione errori (es. email duplicata).
- Endpoint di login con verifica email e password, generazione token JWT firmato e con scadenza.
- Middleware o meccanismo di verifica token JWT per protezione endpoint.
- Endpoint protetti accessibili solo con token JWT valido.
- Endpoint per reset password base, con invio di link temporaneo via email.
- Memorizzazione sicura delle password (hash + salt) e archiviazione utenti in PostgreSQL.
- Logging centralizzato degli eventi di autenticazione (login, reset, errori).
- Conformità a HTTPS e best practice sicurezza per input e token.

## 5. Out of scope
- Social login o OAuth.
- Ruoli o permessi differenziati per utenti.
- Conferma via email per la registrazione.
- Meccanismi avanzati per reset password come autenticazione a due fattori o domande di sicurezza.
- Interfacce di amministrazione o gestione utenti.
- Localizzazione o supporto multilingua.
- Meccanismi anti-brute force o blocchi login non specificati nei requisiti.

## 6. Task tecnici ordinati
1. Progettare schema utenti in PostgreSQL con vincoli univoci su email.
2. Implementare servizio di hashing e salting password.
3. Realizzare endpoint REST per registrazione con validazioni e gestione errori.
4. Implementare endpoint login con verifica credenziali e generazione JWT firmato.
5. Configurare middleware Bear per verifica token JWT su endpoint protetti.
6. Sviluppare endpoint reset password che genera e invia link temporaneo via email.
7. Configurare logging centralizzato per eventi autenticazione.
8. Assicurare protezione delle comunicazioni con HTTPS (configurazione ambiente).
9. Implementare validazione input per prevenzione injection.
10. Predisporre configurazione per scadenza token JWT (es. 1 ora).
11. Test funzionali e di sicurezza per tutti i flussi (registrazione, login, reset, accesso protetto).

## 7. Acceptance criteria
- Un utente può registrarsi usando solo email e password con password memorizzata in modo sicuro.
- Login con email e password restituisce token JWT valido e firmato con scadenza predefinita.
- Endpoint di registrazione, login, reset password sono pubblici; tutti gli altri endpoint sono accessibili solo con token JWT valido.
- Reset password invia correttamente un link di reset temporaneo all’email indicata senza rivelare se l’email è registrata o no.
- Password deboli o email duplicate generano errori espliciti ma non dettagli sulla sicurezza.
- Tutte le comunicazioni avvengono via HTTPS.
- Logging centralizzato registra eventi di autenticazione senza esporre dati sensibili.
- Validazioni input impediscono injection e altri attacchi comuni.
- Il sistema scala orizzontalmente mantenendo la validità dei token JWT.
- Nessun supporto per social login, ruoli, conferma email o reset avanzato è presente.

## 8. Rischi e punti aperti
- Mancanza di meccanismi anti-brute force potrebbe esporre a tentativi di attacco di forza bruta.
- La gestione della revoca del token JWT non è definita, quindi il logout remoto o revoca forzata non è possibile.
- Non è definito il catalogo completo degli endpoint da proteggere, si assume che sia tutto eccetto registrazione, login e reset password.
- Assenza di conferma email potrebbe comportare registrazioni con email errate o non valide.
- Potenziali problemi di sincronizzazione oraria in ambienti distribuiti per validità token JWT.
- Dettagli di configurazione per invio email (smtp, provider) non specificati e da definire.
- Logging centralizzato deve essere progettato con attenzione per rispettare la privacy e conformità GDPR.