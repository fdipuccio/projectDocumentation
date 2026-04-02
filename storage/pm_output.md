MODULE: PM VERSION: 1
## 1. Obiettivo
Realizzare un backend per un sistema di autenticazione utenti che consenta la registrazione, login con email e password, generazione di token JWT per sessioni stateless, protezione di endpoint sensibili tramite autenticazione e un meccanismo base di reset password via email.

## 2. Contesto e vincoli
- Il sistema sarà sviluppato con stack tecnologico Java, REST API, PostgreSQL come database, JWT per gestione token, e Bear come componente da chiarire tra framework o libreria custom.
- Backend esclusivamente, senza frontend o interfaccia utente.
- Sono richiesti meccanismi di sicurezza robusti: hashing sicuro password, protezione token JWT, mitigazione brute force, protezione contro CSRF/XSS.
- Il sistema deve sostenere performance e scalabilità per carichi variabili, con possibilità di scalare orizzontalmente.
- Logging e monitoraggio minimi implementati senza memorizzare informazioni sensibili.
- Dipendenza critica tra funzionalità: login deve precedere generazione token, validazione token necessaria per accesso a endpoint protetti, reset password necessita sistema di invio email configurato.

## 3. Assunzioni
- Dati minimi per registrazione sono soltanto email e password; nessun altro dato utente aggiuntivo previsto.
- Politica di scadenza token JWT impostata a circa 1 ora salvo conferma cliente.
- Reset password implementato tramite invio di link con codice unico temporaneo via email.
- Controllo accesso agli endpoint protetti basato solo su autenticazione (utente autenticato), senza gestione di ruoli o permessi.
- Rate limiting per tentativi di login e reset password è consigliato ma non esplicitamente richiesto, da valutare in fase di implementazione.
- Bear va approfondito per definire modalità integrazione e utilizzo nel sistema.

## 4. Scope MVP
- Implementazione endpoint REST per:
  - Registrazione utente con validazione email/password e controllo unicità email.
  - Login con email/password con generazione e restituzione token JWT.
  - Validazione JWT per accesso a endpoint protetti.
  - Reset password via email con generazione e verifica link/codice temporaneo.
- Gestione sicurezza:
  - Hashing password sicuro (bcrypt o Argon2).
  - Protezione token JWT con secret key robusta e firma.
  - Mitigazione basica per attacchi brute force.
- Logging di accessi, fallimenti login e reset, senza dati sensibili.
- Monitoraggio minimo con metriche e alert su errori critici e anomalie sicurezza.
- Scalabilità orizzontale abilitata per backend.

## 5. Out of scope
- Autenticazione tramite social login o OAuth.
- Gestione avanzata di ruoli o permessi utente.
- Frontend o interfacce utente.
- Notifiche push o sistemi avanzati di auditing oltre logging base.
- Canali di reset password diversi da email.
- Definizione dettagliata di politiche di rate limiting (se non richiesto esplicitamente).
- Funzionalità o configurazioni specifiche relative a Bear non chiarite dal BA.

## 6. Task tecnici ordinati
1. Analisi e definizione finalizzata dell’integrazione Bear.
2. Definizione schema DB PostgreSQL per utenti (email, password hash, dati reset).
3. Implementazione endpoint registrazione con validazione e controllo unicità.
4. Implementazione endpoint login con generazione token JWT con claims e scadenza.
5. Implementazione middleware o filtro per validazione token JWT su endpoint protetti.
6. Realizzazione endpoint reset password con invio email link/codice temporaneo.
7. Configurazione hashing password sicuro (bcrypt/Argon2).
8. Implementazione misure base di rate limiting e mitigazione brute force.
9. Setup logging sicuro e anonimizzato su eventi accesso e reset.
10. Implementazione metriche e alert per monitoraggio sicurezza e errori critici.
11. Test funzionali e di sicurezza (token expiration, concorrenza, edge case).
12. Documentazione tecnica backend e API.

## 7. Acceptance criteria
- Utente può registrarsi con email non duplicata, ricevendo validazione errori formali.
- Utente può effettuare login con combinazione email/password valida e riceve token JWT firmato.
- Accesso a endpoint protetti è consentito solo con token JWT valido e non scaduto.
- Reset password invia link/codice via email e consente cambio password entro scadenza.
- Password memorizzate sono hashate con algoritmi sicuri.
- Sistema previene tentativi di brute force con limitazione ragionevole.
- Logging registra eventi accesso/fallimenti senza esporre dati sensibili.
- Sistema risponde in tempi adeguati sotto carico prevedibile.
- Monitoraggio attiva alert su errori critici o anomalie sicurezza.
- Documentazione backend e API completa, chiara e rilasciata.

## 8. Rischi e punti aperti
- Incertezza sul ruolo preciso e uso di Bear nel progetto, necessaria chiarificazione tecnica.
- Politica definitiva di scadenza token JWT da confermare con stakeholder.
- Implementazione e parametri di rate limiting non definiti con precisione.
- Meccanismo di invio email per reset password dipende da configurazione esterna non dettagliata.
- Possibili limitazioni non previste nello scaling orizzontale da testare in ambiente reale.
- Gestione sicurezza e logging deve evitare esposizione accidentale dati sensibili.
- Mancata definizione di eventuali requisiti futuri su ruoli o permessi che potrebbero richiedere architettura modulare.