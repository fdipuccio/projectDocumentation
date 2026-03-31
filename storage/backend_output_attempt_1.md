MODULE: BACKEND VERSION: 1
## 1. Obiettivo backend
Realizzare un backend API per la gestione dell’autenticazione utenti tramite FastAPI, consentendo registrazione con email e password, login autenticato con generazione di token JWT, protezione di endpoint tramite verifica token e implementazione di una procedura base per il reset password. Il sistema sarà integrabile in architetture più ampie ed utilizzerà PostgreSQL per la persistenza dei dati.

## 2. Assunzioni tecniche
- Il backend è dedicato esclusivamente all’autenticazione e non gestisce ruoli, permessi o autenticazioni esterne.
- Le email degli utenti sono univoche e utilizzate come identificativo principale.
- Le password sono conservate esclusivamente in forma hash sicura tramite algoritmo tipo bcrypt o argon2.
- I token JWT saranno generati utilizzando un segreto e algoritmo standard (es. HS256), senza rotazioni o invalidazioni complesse.
- La procedura di reset password prevede un token temporaneo (random e a scadenza) memorizzato nel DB, senza invio email automatico.
- Non sono implementate funzionalità di sicurezza avanzate come MFA, rate limiting o captcha.
- Il sistema è sviluppato con Python, FastAPI e PostgreSQL utilizzando un ORM (es. SQLAlchemy).
- La documentazione API segue OpenAPI/Swagger generata automaticamente da FastAPI.

## 3. Architettura backend
Il backend segue un’architettura REST API monolitica con FastAPI come framework. La persistenza dati è gestita tramite PostgreSQL, con interazione tramite ORM. La sicurezza e autenticazione è gestita tramite JWT emessi al login e verificati tramite dipendenze FastAPI. I moduli principali si articolano in gestione utenti, autenticazione, protezione endpoint e reset password. Tutta la logica business è separata dai layer di routing e persistenza per garantire testabilità e riusabilità.

## 4. Moduli e responsabilità
- **modello utenti:** definizione schema DB (id, email unica, password hashed, reset_token, reset_token_expiry)
- **database:** setup e connessione PostgreSQL tramite SQLAlchemy o simile ORM
- **security:** hashing password (bcrypt/argon2), generazione e verifica JWT
- **api registrazione:** endpoint POST /register per creazione utente con validazioni
- **api login:** endpoint POST /login per autenticazione e generazione token JWT
- **api protette:** endpoint GET /protected-demo come esempio di endpoint protetto JWT
- **api reset password:** endpoint POST /reset-request (richiesta token reset), POST /reset-confirm (modifica password con token)
- **middleware/dependency:** controllo e validazione token JWT in header Authorization Bearer
- **configurazione:** gestione segreto JWT, durata validità reset token, parametri minimi password

## 5. API principali
- **POST /register**  
  Input: { email, password }  
  Output: conferma registrazione o errore (email duplicata, validazione)  

- **POST /login**  
  Input: { email, password }  
  Output: { access_token: JWT, token_type: "bearer" } o errore (credenziali errate)  

- **GET /protected-demo**  
  Accesso protetto da JWT (header Authorization: Bearer xxx)  
  Output: dati di esempio per utente autenticato  

- **POST /reset-request**  
  Input: { email }  
  Output: conferma generazione token reset (senza esposizione token)  

- **POST /reset-confirm**  
  Input: { email, reset_token, new_password }  
  Output: conferma modifica password o errore (token invalido/scaduto)  

## 6. Business logic
- Alla registrazione, si valida email e password, si controlla unicità email e si salva l’utente con password hashed.
- Al login, si verifica la corrispondenza email/password tramite verifica hash, si genera un JWT con payload minimo (es. user_id).
- La generazione token reset crea un token random e una data di scadenza (es. 1 ora da generazione), salvati sul record utente.
- La conferma reset verifica il token reset associato all’email, controlla scadenza, e aggiorna la password hashed cancellando token reset.
- La protezione endpoint verifica la validità e firma del JWT, estrae dati utente dal payload e permette l’accesso solo se valido.

## 7. Persistenza e integrazioni
- PostgreSQL gestisce la tabella utenti con campi: id (UUID o serial), email (unique), password_hashed, reset_token (nullable), reset_token_expiry (timestamp nullable).
- L’ORM (SQLAlchemy) implementa il modello dati e fornisce interoperabilità sicura verso DB.
- Nessuna integrazione esterna prevista (es. sistemi di invio email), il reset password resta interno e manuale.
- Connessione DB configurata tramite parametri esterni (env variables) e pooling gestito da ORM.

## 8. Autenticazione e autorizzazione
- Autenticazione tramite email e password con hash (bcrypt/argon2).
- Emissione token JWT (HS256) al login, contenenti user_id e timestamp.
- Verifica token JWT tramite dependency FastAPI che decodifica e controlla firma e scadenza (configurata se presente).
- Controlli di autorizzazione limitati: solo verifica utente autenticato, non è previsto controllo ruoli/permessi.
- Reset password prevede token temporaneo memorizzato sul DB come ulteriore verifica prima di modificare la password.

## 9. Gestione errori
- Errori di validazione input (formato email, password minima) restituiti con HTTP 422 Unprocessable Entity.
- Email duplicata a registrazione risponde con 400 Bad Request con messaggio significativo.
- Credenziali errate login rispondono con 401 Unauthorized.
- Accesso a endpoint protetti senza o con token non valido risponde con 401 Unauthorized.
- Token reset password non valido o scaduto risponde con 400 Bad Request con dettaglio.
- Errori di sistema DB restituiti con 500 Internal Server Error (logging interno).
- I messaggi errore sono formattati in JSON con chiave "detail" come da standard FastAPI.

## 10. Strategia di test backend
- Test di integrazione per endpoint principali: registrazione, login, accesso protetto, richiesta e conferma reset password.
- Verifica hashing corretto e non memorizzazione in chiaro.
- Validazione generazione, formato e verifica token JWT.
- Test casi limite: email duplicata, password weak, token reset scaduto e non valido.
- Mock DB in memoria (es. SQLite) per esecuzione test automatizzati.
- Utilizzo di tool di testing FastAPI (TestClient) e framework pytest.

## 11. Rischi tecnici
- Flusso di reset password di base senza invio email può generare problematiche di usabilità o sicurezza.
- Assenza di politiche di scadenza JWT configurate potrebbe facilitare token non aggiornati o riutilizzo.
- Mancanza di protezione avanzata (es. rate limiter) espone il sistema a possibili attacchi brute force.
- Nessuna verifica email, quindi rischio di registrazioni con email non valide o abusive.
- Complessità futura di integrazione in ambienti enterprise con requisiti avanzati di sicurezza non prevista.

## 12. Struttura file proposta
```
/app
  /api
    auth.py             # endpoint register, login, reset password
    protected.py        # esempi endpoint protetti
  /core
    config.py           # lettura config, segreto JWT, parametri reset token
    security.py         # funzioni hashing password, generazione/verifica JWT, reset token
  /models
    user.py             # modello dati User ORM
  /db
    session.py          # setup sessione DB (engine, session factory)
  /dependencies
    auth_dependency.py  # dependency per verifica token JWT e utente
  /tests
    test_auth.py        # test integrazione autenticazione
    test_reset.py       # test reset password
main.py                # entry point FastAPI app
```

## 13. Piano di implementazione
1. Definizione schema DB e modello ORM User con campi richiesti.
2. Setup ambiente FastAPI, connessione a PostgreSQL con configurazioni di base.
3. Implementazione hashing password sicuro (bcrypt o argon2) e funzioni di confronto.
4. Endpoint /register con validazioni email/password e persistenza nuovo utente.
5. Endpoint /login con verifica credenziali e generazione token JWT.
6. Funzioni centralizzate per creazione e verifica token JWT.
7. Dependency FastAPI per protezione endpoint tramite verifica token.
8. Endpoint protetto /protected-demo per test accesso autenticato.
9. Endpoint /reset-request per generazione e memorizzazione token reset.
10. Endpoint /reset-confirm per verifica token reset e modifica password.
11. Copertura test automatizzati per tutte le funzionalità sviluppate.
12. Documentazione API OpenAPI generata automaticamente da FastAPI con descrizioni dettagliate.
13. Revisione codice e test finali prima rilascio.