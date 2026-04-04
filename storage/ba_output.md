MODULE: BA VERSION: 1

## 1. Requisiti funzionali

1.1 Ricezione ordini
- Il sistema backend deve esporre API REST per la ricezione di nuovi ordini da parte di client e-commerce.
- Deve validare i dati dell’ordine in ingresso (es. formato, completezza, referenze prodotto).
  
1.2 Integrazione gateway pagamento (Stripe)
- Supporto per autorizzazione (authorization) e acquisizione fondi (capture) tramite Stripe API.
- Gestione stati pagamento correlati all’ordine (autorizzato, catturato, fallito).
- Retry automatico per errori di comunicazione transitori con Stripe.
- Operazioni idempotenti per evitare duplicazione transazioni in retry.

1.3 Workflow ordine
- Stati principali: validazione → pagamento → conferma ordine → creazione spedizione → spedizione completata.
- Transizioni di stato devono essere tracciate in audit log.
- Ogni stato deve produrre eventi asincroni per comunicazione ad altri microservizi.

1.4 Consumer asincrono aggiornamenti stato
- Consumer di messaggi da RabbitMQ per aggiornamenti di stato ordine provenienti da altri servizi.
- Aggiornamento locale dello stato ordine e gestione conseguenze (es. notifica cliente).

1.5 Integrazione servizio logistica
- Chiamata a servizio esterno per creazione spedizioni e recupero stato tracking.
- Retry automatico in caso di errori transitori.
- Idempotenza delle richieste per evitare spedizioni duplicate.

1.6 Notifiche cliente
- Invio di email e notifiche push a ogni variazione dello stato ordine.
- Integrazione con SendGrid per email.
- Possibilità di configurare template e contenuti notifiche.

1.7 API backoffice operatori
- API REST riservate a operatori per:
  - Visualizzazione lista ordini con filtri e paginazione.
  - Visualizzazione dettaglio singolo ordine.
  - Esecuzione rimborso manuale ordini.
- Autenticazione e autorizzazione tramite JWT.

1.8 Gestione magazzino
- Scalare stock articoli a pagamento confermato.
- Ripristinare stock in caso di cancellazione o rimborso.
- Garantire consistenza e aggiornamenti atomici del magazzino in transazioni ACID.

## 2. Requisiti non funzionali

2.1 Sicurezza
- Autenticazione e autorizzazione JWT per API pubbliche e backoffice.
- Validazione e sanificazione input per prevenire injection e vulnerabilità.
- Crittografia dati sensibili (es. dati pagamento) in transito e a riposo.
- Audit log per tutte le modifiche di stato ordine, accessi e operazioni critiche.

2.2 Performance
- Risposta API REST entro livelli accettabili (es. <200ms per richieste standard).
- Capacità di elaborare picchi di ordini simultanei.
- Consumer asincrono gestito con buffering e backpressure da RabbitMQ.

2.3 Scalabilità
- Architettura microservizio autonomo, deployabile su cluster/container.
- Scalabilità orizzontale tramite instanze multiple.
- Utilizzo di code RabbitMQ per decoupling e load balancing.

2.4 Affidabilità / resilienza
- Retry automatici su operazioni di payment e logistica in caso di errori temporanei.
- Operazioni idempotenti per sicurezza contro duplicazioni in retry.
- Gestione transazioni ACID su PostgreSQL per garantire consistenza dati.

2.5 Logging e monitoraggio
- Logging centralizzato dei flussi di ordine, errori e transizioni.
- Metriche di performance e health check per monitoring continuo.
- Tracciabilità completa degli eventi e operazioni per audit e troubleshooting.

## 3. Ambiguità e domande aperte

3.1 Campo ordini ricevuti via API:
- Ambiguità: quali dati esatti deve contenere un ordine (es. indirizzo, prodotti, pagamenti parziali)?
- Assunzione: il payload deve contenere informazioni complete su cliente, prodotti, quantità, indirizzo spedizione, e dati pagamento.

3.2 Stato pagamento: differenza gestione authorization vs capture 
- Ambiguità: se deve supportare pagamento separato o solo pagamento immediato.
- Assunzione: supporto completo a flusso authorization + capture distinto.

3.3 Handling rimborsi manuali
- Ambiguità: modalità e regole di rimborso manuale (es. partiale, totale, tempistiche).
- Assunzione: backend espone API per rimborso totale; regole di business da definire fuori da questo scope.

3.4 Dettaglio gestione scorte
- Ambiguità: gestione magazzino per ordini multipli con stock limitato.
- Assunzione: locking e verifica preventiva per evitare overselling.

3.5 Dettagli integrazione notifiche push
- Ambiguità: quale sistema push sarà usato oltre a SendGrid per email? Canali supportati?
- Assunzione: sistema push generico configurabile; dettagli da definire.

3.6 Livello di dettaglio audit log
- Ambiguità: quali campi esatti devono essere tracciati e per quanto tempo conservare i log.
- Assunzione: tracciare tutte le transizioni di stato, utenti che operano, timestamp con conservazione conforme policy GDPR.

3.7 Autenticazione API operatori
- Ambiguità: esiste un sistema utenti esterno o va implementato internamente?
- Assunzione: esiste sistema di autenticazione esterno integrato via JWT.

## 4. Edge case e scenari limite

4.1 Ordine con pagamenti parziali o multipli (split payment)
4.2 Retry illimitato o molto frequente su chiamate fallite (gestire backoff e circuit breaker)
4.3 Concorrenza su magazzino per ordini simultanei dei medesimi prodotti
4.4 Messaggi duplicati o out-of-order da queue asincrona
4.5 Fallimento del servizio esterno di logistica per lunghi periodi (degradazione servizio)
4.6 Gestione ordini cancellati a vari stadi del workflow (pre e post pagamento)
4.7 Notifiche inviate duplicatamente o che non arrivano al cliente
4.8 Integrazione multi-tenant o supporto multi-store (non specificato, ma possibile requisito futuro)

## 5. Dipendenze e vincoli

5.1 Dipendenze funzionali
- Validazione ordine → pagamento → conferma ordine → spedizione: sequenza obbligata.
- Gestione magazzino dipende da conferma pagamento o cancellazione.
- Notifiche dipendono dalla transizione di stato ordine.
- Consumer RabbitMQ per aggiornamenti stato deve essere sincronizzato con stato interno.

5.2 Vincoli tecnici
- Persistenza su PostgreSQL con transazioni ACID.
- Comunicazione asincrona con altri microservizi tramite RabbitMQ.
- Operazioni critiche devono essere idempotenti.
- Resilienza garantita da retry automatici per servizi esterni (Stripe, logistica).
- Deploy microservizio indipendente in ambiente container.

5.3 Vincoli di sicurezza
- Utilizzo di JWT per autenticazione API.
- Protezione delle API da accessi non autorizzati, validazione input.

## 6. Requisiti esclusi esplicitamente

- Non è richiesta la gestione frontend o interfacce utente grafiche.
- Non è richiesto lo sviluppo del sistema di autenticazione utenti (assunto esterno).
- Non sono previsti sistemi di pagamento diversi da Stripe.
- Non è previsto supporto per multicanalità vendita (es. negozi fisici).
- Non è richiesto sviluppo del servizio di logistica (solo integrazione).
- Non è richiesto il supporto a valute multiple o lingue.
- Non sono richiesti report avanzati o analisi business intelligence.
- Non è prevista gestione manuale delle code RabbitMQ (solo consumer automatico).

---

Questo documento chiarisce e struttura i requisiti attuali, evidenziando punti da approfondire con il cliente e anticipando problematiche tecniche e di progetto.