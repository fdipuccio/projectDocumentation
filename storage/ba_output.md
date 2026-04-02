MODULE: BA VERSION: 1
## 1. Requisiti funzionali
- Registrazione utente: deve essere possibile creare un nuovo utente fornendo dati minimi (tipicamente email e password). Implicito: validazione formale di email e password, verifica unicità email.
- Login con email e password: sistema di autenticazione che consente agli utenti di effettuare il login usando combinazione email/password.
- Generazione token JWT: dopo login valido, deve essere generato un token JWT per mantenere lo stato di sessione lato client. Implicito: definizione di scadenza del token, claim minimi da includere nel token.
- Endpoint protetti: alcune API devono richiedere un token JWT valido per l’accesso. Implicito: gestione decorso e revoca token, validazione token lato backend.
- Reset password (base): funzione per reimpostare password dimenticata o compromessa. Implicito: meccanismo per inviare link o codice per reset, limitazioni/rate limiting per evitare abusi.

## 2. Requisiti non funzionali
- Sicurezza: hashing sicuro delle password (es. bcrypt / Argon2), protezione da attacchi brute force, gestione sicura del token JWT (secret key robusta, firma), protezione CSRF/XSS sui servizi REST.
- Performance: il backend deve sostenere un carico di richieste di autenticazione e autorizzazione in modo efficiente (target da definire).
- Scalabilità: il sistema deve poter scalare orizzontalmente per gestire un aumento degli utenti senza compromettere performance e sicurezza.
- Logging: tracciamento sicuro degli accessi, fallimenti di login, reset password, con attenzione a non loggare dati sensibili come password o JWT.
- Monitoraggio: implementazione di metriche e alert su errori critici e anomalie di sicurezza.

## 3. Ambiguità e domande aperte
- Che dati utente devono essere raccolti in fase di registrazione oltre a email e password? Assumiamo solo email e password come minimo.
- Qual è la politica di scadenza del token JWT? Si assume una scadenza ragionevole (es. 1 ora) ma da validare con il cliente.
- Come deve essere gestito il reset password? Solo via email o anche altri canali? Assumiamo invio di link via email con codice unico temporaneo.
- Gli endpoint protetti devono prevedere ruoli o permessi? Nessun requisito esplicito, si assume controllo base “utente autenticato” senza livelli.
- Vi sono limiti di frequenza per autenticazione e reset password? Non definito, si consiglia rate limiting per sicurezza.

## 4. Edge case e scenari limite
- Tentativi multipli falliti di login e reset password (brute force).
- Token JWT scaduto o invalidato e tentativo di accesso a endpoint protetti.
- Registrazione con email già esistente.
- Reset password con link scaduto o già utilizzato.
- Concorrenza simultanea di richieste di login e reset per lo stesso utente.

## 5. Dipendenze e vincoli
- Dipendenza tra login e generazione token JWT: token deve essere generato solo dopo login successo.
- Endpoint protetti dipendono dalla corretta validazione del token JWT.
- Reset password dipende da sistema di invio email configurato.
- Vincolo tecnico: stack Java con REST API, database PostgreSQL, utilizzo JWT per autenticazione stateless.
- Bear è parte dello stack ma non è chiaro se è framework o libreria custom, va chiarito per definire integrazione.

## 6. Requisiti esclusi esplicitamente
- Non è richiesto supporto per autenticazione con social login o OAuth.
- Non è prevista gestione di ruoli o permessi avanzati.
- Non è richiesto frontend o interfaccia utente, solo backend.
- Non sono previste notifiche push o sistemi di auditing avanzati oltre logging di base.