MODULE: PM VERSION: 1
## 1. Obiettivo
Realizzare un backend per la gestione dell’autenticazione utenti che consenta la registrazione, login tramite email e password, generazione e validazione di token JWT, protezione di endpoint REST API e gestione base del reset password, utilizzando lo stack tecnologico Java, REST API, Bear framework, PostgreSQL e JWT.

## 2. Contesto e vincoli
- Lo sviluppo deve avvenire in ambiente enterprise con particolare attenzione alla sicurezza, performance (<200ms per login/registrazione) e scalabilità orizzontale.
- Persistenza dati utenti tramite PostgreSQL.
- Architettura modulare backend integrata con Bear framework.
- Utilizzo di JWT per autenticazione e autorizzazione alle API protette.
- Gestione password con hashing e salting (es. bcrypt).
- Dipendenza esterna per invio email nel processo di reset password.
- Nessuna funzionalità di multi-fattore o ruoli avanzati è prevista.
- Comunicazioni tra client e server su canali sicuri HTTPS.

## 3. Assunzioni
- Policy password: minima 8 caratteri con lettere e numeri (senza definizione esplicita nel requisito originale).
- Validità token JWT impostata a 1 ora.
- Token per reset password con validità di 15 minuti.
- Invio email di reset password tramite sistema esterno integrato.
- Conferma email per attivazione account è opzionale e da valutare.
- Sistema iniziale senza gestione di ruoli/autorizzazioni multiple.
- Protezione da attacchi comuni implementata a livello applicativo (es. blocco login dopo tentativi multipli falliti, mitigazione CSRF).
- Logging e monitoraggio limitati agli eventi di autenticazione e reset password.

## 4. Scope MVP
- Endpoint per registrazione utente con validazione email e policy password.
- Endpoint login con verifica credenziali e generazione token JWT.
- Middleware o filtro per protezione endpoint tramite verifica token JWT.
- Endpoint per richiesta reset password con generazione e invio token temporaneo.
- Endpoint per cambio password utilizzando token di reset.
- Persistenza dati utenti e token di reset in PostgreSQL.
- Meccanismi base di sicurezza (hashing password, trasmissione HTTPS).
- Logging eventi critici di autenticazione e reset.

## 5. Out of scope
- Autenticazione multi-fattore (2FA).
- Gestione di ruoli e permessi avanzati o livelli di autorizzazione differenti.
- Integrazione con provider esterni per login social o SSO.
- Funzionalità di logout esplicito e revoca token JWT.
- Registrazione utente con dati oltre email e password.
- Personalizzazioni avanzate del flusso di reset password (oltre il livello base).
- Conferma obbligatoria con email prima dell’attivazione dell’account (proposta opzionale non implementata nel MVP).

## 6. Task tecnici ordinati
1. Definizione schema dati utenti, token reset e log eventi in PostgreSQL.
2. Implementazione endpoint REST per registrazione utente con validazioni (email, password).
3. Implementazione endpoint login con verifica credenziali e creazione token JWT.
4. Configurazione Bear framework per gestione token JWT e filtro autorizzazione su endpoint protetti.
5. Implementazione endpoint reset password: richiesta token, invio email tramite sistema esterno.
6. Implementazione endpoint di aggiornamento password con validazione token reset.
7. Implementazione hashing e salting password con bcrypt o equivalente.
8. Logging audit eventi login, reset e errori di autenticazione.
9. Gestione errori e edge case (tentativi login falliti multipli, token scaduti o manomessi).
10. Configurazione HTTPS e audit sicurezza delle comunicazioni.
11. Testing funzionale e di sicurezza (include test performance).

## 7. Acceptance criteria
- Successo della registrazione con utenti email univoci e password conformi policy.
- Login restituisce token JWT valido con scadenza di 1 ora.
- Endpoint protetti rifiutano richieste senza token valido.
- Possibilità di richiesta reset password con generazione e invio token temporaneo valido 15 minuti.
- Cambio password accettato solo con token reset valido.
- Password memorizzate solo in forma hashed e salted.
- Tempi di risposta login/registrazione inferiori a 200ms in condizioni di carico normale.
- Logging di eventi critici presente e consultabile.
- Gestione appropriata di errori e casi limite come token scaduto, login multipli falliti.

## 8. Rischi e punti aperti
- Mancata definizione definitiva della policy password e gestione conferma email possono impattare sicurezza.
- Dipendenza da sistema esterno per l’invio email di reset password potrebbe introdurre vulnerabilità o ritardi.
- Non gestita esplicitamente la revoca token JWT o logout esplicito (potenziale rischio di token compromessi).
- Possibili problemi di sicurezza avanzata (es. attacchi replay token) da approfondire e mitigare.
- Necessità di definire e implementare meccanismi di blocco/tolleranza per tentativi login falliti.
- Ambiguità su gestione più complessa di ruoli o livelli autorizzativi in futuro.
- Monitoraggio e alerting solo parzialmente definito, potrebbe essere necessario estenderlo per produzione.