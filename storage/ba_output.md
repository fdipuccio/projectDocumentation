MODULE: BA VERSION: 1

## 1. Requisiti funzionali
1.1. Ricezione ordini tramite API REST
- Il sistema deve esporre API REST sicure per ricevere ordini da client esterni o altri moduli.
- Deve includere validazione degli input ordini (esistenza, formato, dati obbligatori).
- Gestione eventuali ordini duplicati (idempotenza su creazione).

1.2. Integrazione con gateway di pagamento Stripe per authorization e capture
- Chiamata a Stripe per autorizzare pagamenti al momento opportuno nel workflow.
- Successiva cattura (capture) dei fondi dopo validazione.
- Retry automatico in caso di errori transitori.
- Idempotenza nell'invocazione API verso Stripe per evitare multipli addebiti.

1.3. Workflow di elaborazione ordine
- Stati definiti almeno: Validazione → Pagamento → Conferma → Spedizione.
- Transizioni di stato devono essere tracciate con audit log.
- Validazione dati ordine e stock prima del pagamento.
- Conferma ordine solo a pagamento eseguito con successo.

1.4. Consumer asincrono per eventi da RabbitMQ
- Consumo asincrono di eventi di aggiornamento stato ordine.
- Gestione di eventuali duplicati o eventi fuori sequenza.

1.5. Integrazione con servizio esterno di logistica
- Creazione spedizioni e gestione tracking spedizioni tramite API esterne.
- Retry automatico in caso di fallimenti transitori.
- Aggiornamento stato ordine in base a risposte logistiche.

1.6. Notifiche email/push tramite SendGrid
- Invio notifiche al cliente per ogni cambio di stato dell’ordine.
- Supporto notifiche email e notifiche push (canali da definire).
- Gestione eventuali fallimenti di invio con retry.

1.7. API di backoffice per operatori
- Visualizzazione lista ordini.
- Visualizzazione dettaglio ordine.
- Funzionalità di rimborso manuale (con integrazione pagamento?).
- Controlli di autorizzazione e autenticazione operatori (es. JWT).

1.8. Gestione magazzino
- Scalare stock al momento di pagamento confermato.
- Ripristinare stock in caso di cancellazione ordine o errore.
- Verifica disponibilità stock prima di procedere al pagamento.

## 2. Requisiti non funzionali
2.1. Sicurezza
- Autenticazione e autorizzazione via JWT sulle API REST.
- Validazione e sanificazione input per prevenire injection e vulnerabilità.
- Protezione dati sensibili (ad es. informazioni pagamento e clienti).
- Audit log completo e immutabile per tracciare tutte le transizioni ordine.

2.2. Performance
- Risposta API REST entro tempi accettabili anche sotto carico.
- Gestione efficiente dei retry su chiamate esterne per minimizzare latenza percepita.

2.3. Scalabilità
- Architettura microservizi scalabile orizzontalmente.
- Code RabbitMQ per decomporre flussi di lavoro asincroni.

2.4. Affidabilità e resilienza
- Retry automatico su errori transitori con limiti e backoff.
- Operazioni critiche idempotenti per evitare duplicazioni in caso di retry.
- Gestione robusta delle transizioni di stato e consistenza dati.

2.5. Logging e monitoraggio
- Logging strutturato di tutte le operazioni critiche.
- Monitoraggio stato code, operazioni pagamento, notifiche.
- Alerting su errori critici e degradazione servizi esterni.

## 3. Ambiguità e domande aperte
- Dettaglio esatto degli eventi consumati da RabbitMQ: quali eventi e struttura? Assumo eventi di stato ordine.
- Canali e formati delle notifiche push non specificati: assumo email con possibilità futura di estendere su mobile push.
- Dettaglio API backoffice e permessi operatori: chi gestisce gli operatori? Assumo ruolo base di amministratore per rimborso.
- Strategia retry e gestione errori per Stripe e logistica solo abbozzata, serve dettaglio su numero tentativi e intervalli.
- Gestione cancellazione ordine non dettagliata: chi può cancellare? È automatica o manuale? Assumo cancellazione operatori e/o predefinita.
- Tempo di validità autorizzazione pagamento Stripe implicito ma non esplicitato.
- Compliance normativa (es. GDPR, PCI-DSS) non menzionata ma rilevante essendo dati sensibili e pagamenti.

## 4. Edge case e scenari limite
- Ricezione doppia richiesta identica di ordine (es. retry network): idempotenza da gestire.
- Pagamenti autorizzati ma non catturati: cosa succede? Timeout o processi di cleanup.
- Situazioni di stock insufficiente tra validazione e pagamento.
- Fault parziale in integrazione logistica: spedizione creata ma ordine non aggiornato.
- Down temporaneo di SendGrid o Stripe: gestione code e retry.
- Concorrenza su aggiornamenti stati ordine, possibili race condition.
- Gestione eventi fuori sequenza o duplicati dalla queue.

## 5. Dipendenze e vincoli
- Workflow ordine dipende da integrazione Stripe (pagamento) e logistica (spedizione).
- Gestione stock dipende da conferma pagamento.
- Notifiche dipendono da transizioni stato ordine.
- Microservizio indipendente ma deve integrarsi tramite event queue RabbitMQ con altri microservizi.
- PostgreSQL scelto per garantire transazioni ACID, necessario per consistency dati.
- JWT per sicurezza API, quindi dipendenza da sistema di identity management.
- Retry e idempotenza come vincoli trasversali sulle chiamate esterne.

## 6. Requisiti esclusi esplicitamente
- Non è richiesto sviluppo di frontend cliente o backoffice (solo API).
- Non è prevista integrazione con altri gateway di pagamento diversi da Stripe.
- Non è richiesto sistema di pagamento diretto, solo integrazione con Stripe esistente.
- Non è richiesto sistema di gestione utenti o ruoli oltre JWT per operazioni backoffice.
- Non è richiesto sistema di inventario complesso oltre scalare/ripristino stock.
- Non sono richiesti report avanzati o analisi ordini.
- Non è prevista gestione multi-valuta o multi-paese esplicitamente.
- Non sono previsti canali di notifica diversi da email/push tramite SendGrid.