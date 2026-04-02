MODULE: BA VERSION: 1
## 1. Requisiti funzionali
- Registrazione utente:
  - Deve permettere la creazione di un nuovo utente con email e password.
  - Validazione dell'email (formato corretto, unicità).
  - Password con policy di complessità da definire (es. lunghezza minima, caratteri speciali).
- Login con email e password:
  - Autenticazione dell'utente tramite email e password.
  - Verifica correttezza delle credenziali.
- Generazione token JWT:
  - Creazione di un token JWT al momento del login con scadenza definita.
  - Il token deve contenere le informazioni minimal necessarie per l’identificazione e l’autorizzazione.
- Endpoint protetti:
  - Implementazione di meccanismi di autorizzazione che richiedono token JWT valido.
  - Protezione di tutte le API che necessitano autenticazione.
- Reset password (base):
  - Permettere all’utente di richiedere un reset password tramite email.
  - Generazione e validazione di un token temporaneo per reset password.
  - Possibilità per utente di impostare una nuova password utilizzando il token.

## 2. Requisiti non funzionali
- Sicurezza:
  - Password memorizzate solo in forma hashed e salted (es. bcrypt).
  - Protezione da attacchi comuni (brute force, injection, CSRF).
  - Trasmissione dati su canali sicuri (HTTPS).
- Performance:
  - Il sistema deve garantire tempi di risposta adeguati (<200ms per login/registrazione).
- Scalabilità:
  - Architettura backend modulare, consente scalabilità orizzontale.
  - Supporto per numero elevato di utenti concorrenti.
- Logging e monitoraggio:
  - Audit log di eventi di autenticazione e reset password.
  - Monitoraggio degli endpoint per rilevare tentativi sospetti.
 
## 3. Ambiguità e domande aperte
- Policy password non definita (lunghezza minima, requisiti complessità): assumiamo almeno 8 caratteri con lettere e numeri.
- Durata validità token JWT non specificata: assumiamo 1 ora come default.
- Come viene gestita la scadenza del token di reset password? Proponiamo 15 minuti.
- L’endpoint di reset password invia una email? Non è esplicitato; ipotizziamo invio email tramite sistema esterno.
- Non è indicato se la registrazione prevede conferma email per attivazione account: proponiamo conferma opzionale da valutare.
- Non è espresso se ci siano ruoli o livelli di autorizzazione diversi: assumiamo sistema base senza gestione ruoli.
  
## 4. Edge case e scenari limite
- Tentativi multipli di login falliti (possibile blocco temporaneo account).
- Gestione di token JWT scaduti o manomessi.
- Reset password richiesto più volte in breve tempo.
- Registrazione con email già esistente.
- Cambio password con token scaduto.
- Accesso a endpoint protetti senza token o con token non valido.
- Possibili attacchi di replay sul token.
  
## 5. Dipendenze e vincoli
- Dipendenza forte tra login, generazione token JWT e protezione endpoint.
- Reset password dipende da sistema di invio email esterno (non specificato ma implicito).
- Vincolo tecnico: utilizzo di Java, REST API, PostgreSQL, JWT e framework Bear impone scelte precise su librerie e implementazione.
- Architettura backend deve integrare PostgreSQL per persistenza dati utente.
  
## 6. Requisiti esclusi esplicitamente
- Non sono richiesti:
  - Autenticazione multi-fattore (2FA).
  - Gestione ruoli e permessi avanzati.
  - Social login o integrazione con provider esterni (es. OAuth, SSO).
  - Funzionalità di logout esplicito e revoca token JWT.
  - Registrazione con dati aggiuntivi oltre email e password.
  - Personalizzazione avanzata del processo di reset password oltre il livello base.