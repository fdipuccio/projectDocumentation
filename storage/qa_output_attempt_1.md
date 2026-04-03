MODULE: QA
VERSION: 1
FINAL_STATUS: APPROVED_WITH_CHANGES

## 1. Checklist — Copertura requisiti
* [SI] Un utente può registrarsi usando solo email e password con password memorizzata in modo sicuro — La proposta backend prevede hashing e salting password, email unica, validazione formale e gestione errori (409 email duplicata).
* [SI] Login con email e password restituisce token JWT valido e firmato con scadenza predefinita — JwtService genera token firmato con durata configurabile, login con verifica credenziali è previsto.
* [SI] Endpoint di registrazione, login, reset password sono pubblici; tutti gli altri endpoint sono accessibili solo con token JWT valido — Chiaramente definito in proposta con middleware di protezione e elenco endpoint pubblici.
* [SI] Reset password invia correttamente un link di reset temporaneo all’email indicata senza rivelare se l’email è registrata o no — EmailService con invio link reset temporaneo e risposta generica è specificato.
* [SI] Password deboli o email duplicate generano errori espliciti ma non dettagli sulla sicurezza — Validazione input con messaggi generici e gestione 409 per duplicati sono specificati.
* [SI] Tutte le comunicazioni avvengono via HTTPS — HTTPS obbligatorio è assunto con configurazione ambiente menzionata.
* [SI] Logging centralizzato registra eventi di autenticazione senza esporre dati sensibili — LoggingService con anonimizzazione e conformità GDPR prevista.
* [SI] Validazioni input impediscono injection e altri attacchi comuni — InputValidator con validazioni su email e password è indicato.
* [SI] Il sistema scala orizzontalmente mantenendo la validità dei token JWT — Architettura stateless con JWT e durata fissa senza revoca è confermata.
* [SI] Nessun supporto per social login, ruoli, conferma email o reset avanzato è presente — Esplicitamente esclusi e assenti nella proposta.

## 2. Checklist — Contratti API
* [SI] Ogni endpoint ha metodo HTTP, path, request schema, response schema e esempio concreto — Post /register, /login, /reset-password e protetto GET sono completamente descritti.
* [SI] Le regole di validazione sono esplicite per ogni campo (tipo, formato, obbligatorietà, regex ove necessario) — Email RFC valida, password con minimo 8 caratteri, obbligatorietà chiara.
* [SI] Il formato degli errori è consistente tra tutti gli endpoint (struttura JSON uniforme) — Errori sono JSON con campo "error" nelle risposte 4xx, coerenza evidenziata.
* [SI] I codici HTTP di risposta (2xx, 4xx, 5xx) sono specificati per ogni endpoint — 201, 400, 409, 200, 401 sono usati consistentemente e documentati.
* [NO] La strategia di paginazione è definita per le liste (se applicabile) — Non applicabile perché non vi sono endpoint di lista o paginazione nel perimetro.

## 3. Checklist — Business logic e scenari limite
* [SI] I flussi principali sono descritti passo per passo (non solo a parole generiche) — Flussi di registrazione, login, reset password e middleware per JWT sono dettagliati con step precisi.
* [PARZIALE] Gli scenari di concorrenza sono trattati (es. doppia registrazione, doppio click, race condition) — Verifica e gestione email duplicata prevista, ma mancata menzione esplicita di gestione race condition atomica o lock DB.
* [NO] I casi di fallimento delle integrazioni esterne hanno una strategia (retry, fallback, circuit breaker) — Non sono riportate strategie di gestione errori per SMTP o DB in caso di fallimento.
* [SI] Le regole di business critiche sono esplicite e non ambigue — Regole chiare su validazione, scadenza token, endpoint protetti e pubblici, errori generici.

## 4. Checklist — Persistenza e schema dati
* [SI] Le tabelle/collezioni principali sono definite con i campi e i tipi — Tabella utenti con id, email unica, password_hash, salt, created_at, updated_at specificati.
* [SI] Gli indici sono specificati per le colonne usate in query frequenti o join — Email vincolata unica implicitamente indice.
* [PARZIALE] I vincoli di unicità e foreign key sono dichiarati — Vincolo unique su email è menzionato, ma non sono presenti foreign key né discussa loro gestione (ma probabilmente non applicabile).
* [NO] La strategia di migrazione dello schema è menzionata — Non è menzionata alcuna strategia di migrazione/versioning schema DB.

## 5. Checklist — Strategia di test
* [SI] Esistono test per i happy path di ogni funzionalità principale — Test di registrazione, login, reset password con esito positivo definiti.
* [SI] Esistono test per i casi di errore critici (validazione fallita, not found, unauthorized) — Test su email duplicata, invalidi credenziali, assenza token sono inclusi.
* [SI] Esistono test di integrazione per le dipendenze esterne (DB, servizi esterni) — Test integrati con DB e mock invio email sono descritti.
* [SI] I test specificano input e expected output concreti (non generici) — Test con input JSON ed esiti HTTP concreti riportati.

## 6. Requisiti mancanti
- Mancanza di strategia di gestione errori per integrazione fallita con SMTP (retry, fallback).
- Mancanza di strategia per migrazione schema DB.
- Assenza di strategie esplicite per condizioni di concorrenza oltre il controllo unicità email.

## 7. Rischi e problemi
- [ALTA] Assenza di meccanismi anti-brute force espone a rischio attacchi forza bruta su login.
- [ALTA] Mancata revoca token JWT può consentire accessi non autorizzati in caso di token compromessi.
- [MEDIA] Mancata gestione o indicazione di strategie su fallimenti integrazioni esterne (SMTP) può causare malfunzionamenti funzionalità reset password.
- [MEDIA] Assenza strategia migrazione schema rischia downtime o inconsistenza nel deploy su DB evoluti.
- [BASSA] Limitata trattazione concorrenza oltre controllo unicità potrebbe causare rare condizioni di errori o duplicati.

## 8. Azioni richieste
[PRIORITÀ ALTA] Definire e implementare una strategia di gestione errori per integrazione SMTP con retry o fallback per garantire affidabilità invio email reset password.  
[PRIORITÀ ALTA] Integrare un piano di migrazione schema DB per gestione evolutiva strutture dati in produzione.  
[PRIORITÀ MEDIA] Documentare meglio le strategie di gestione concorrenza, inclusa possibile prevenzione race condition nella registrazione utente.  
[PRIORITÀ BASSA] Valutare e proporre meccanismi anti-brute force (anche se out of scope PM, evidenziare rischio alto).  
[PRIORITÀ BASSA] Considerare la gestione di revoca token in linee guida future o patch successive.