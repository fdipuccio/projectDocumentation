MODULE: BA VERSION: 1

## 1. Requisiti funzionali
- Ricezione ordini tramite API REST:
  - Endpoint per creare un ordine e ricevere i dati necessari (cliente, prodotti, quantità, indirizzo, ecc.)
  - Validazione degli input lato backend sulla correttezza e completezza dei dati.
- Integrazione con gateway di pagamento Stripe:
  - Autorizzazione del pagamento (authorization).
  - "Capture" dell'importo dopo conferma ordine.
  - Gestione eventuali errori o rifiuti da Stripe.
- Workflow di elaborazione ordine con stato e transizioni:
  - Stati tipici: validazione, pagamento, conferma, spedizione.
  - Ogni transizione deve essere registrata (audit log).
  - Gestione rollback o flow alternativo in caso di errore.
- Consumer asincrono per eventi da RabbitMQ:
  - Ricezione di eventi di aggiornamento stato ordine da queue.
  - Elaborazione asincrona affidabile e idempotente di tali eventi.
- Integrazione con servizio esterno di logistica:
  - Creazione spedizioni.
  - Aggiornamento tracking shipment.
  - Gestione errori e retry.
- Notifiche a clienti:
  - Email e push notification.
  - Trigger a ogni cambio di stato ordine.
  - Integrazione con SendGrid per l'invio mail.
- API di backoffice per operatori:
  - Lista ordini.
  - Dettaglio ordine.
  - Operazione di rimborso manuale.
- Gestione magazzino:
  - Scalare stock al pagamento confermato.
  - Ripristinare stock in caso di cancellazione ordine.
- Idempotenza:
  - Tutte le operazioni critiche devono prevenire duplicazioni in caso di retry (ad es. payment, stock update).
- Audit log:
  - Tracciare tutte le transizioni di stato ordine e operazioni significative.

## 2. Requisiti non funzionali
- Sicurezza:
  - Autenticazione e autorizzazione JWT per API, specialmente backoffice.
  - Validazione stringente degli input per prevenire injection e vulnerabilità.
  - Protezione dati sensibili (es. dati pagamento, informazioni cliente).
- Performance:
  - Sistemi di caching dove opportuno per ridurre latenza richieste frequenti backoffice.
  - Ottimizzazione query PostgreSQL e gestione connessioni.
- Scalabilità:
  - Microservizio autonomo e deployabile indipendentemente.
  - Capacità di scalare in orizzontale per gestire picchi di ordini.
- Resilienza e affidabilità:
  - Retry automatico su fallimenti transitori (payment, logistica).
  - Gestione errori e fallback degli eventi da queue.
  - Comunicazione asincrona tramite eventi per decoupling.
- Logging e monitoraggio:
  - Audit log di stato ordini.
  - Log errori e warning per troubleshooting.
  - Monitoraggio health service, throughput richieste, latenza.
- Consistenza:
  - Utilizzo di transazioni ACID su PostgreSQL per operazioni critiche (es. pagamento, stock).
- Idempotenza:
  - Meccanismi software per prevenire effetti duplicati conseguenti a retry o eventi ripetuti.

## 3. Ambiguità e domande aperte
- Dettaglio flusso di rimborso manuale: 
  - Il rimborso riguarda solo importo pagamento o include anche il ripristino stock? Assunzione: deve gestire entrambi.
- Cosa succede se il servizio di logistica non è disponibile:
  - Va previsto un meccanismo di coda o retry? Assunzione: sì, retry automatico previsto da requisito resilienza.
- Modalità e frequenza delle notifiche push oltre email:
  - Sono push mobile app o web push? Assunzione: push generiche da integrare tramite SendGrid o altro servizio.
- Gestione concorrente degli ordini sullo stesso stock:
  - Serve un lock pessimista o ottimistico per la scalatura stock? Assunzione: gestione tramite transazioni ACID.
- Livello di dettaglio dei dati nell’audit log:
  - Solo stati ordine o anche dati modificati? Assunzione: audit dettagliato delle transizioni di stato.
- Ambito e autorizzazioni dell’API backoffice:
  - Solo operatori interni o anche partner esterni? Assunzione: operatori interni aziendali.
- Gestione di ordini parzialmente spediti o parzialmente rimborsati:
  - Non specificato, ipotizzare gestione semplicistica o future estensioni?

## 4. Edge case e scenari limite
- Ordini con pagamenti falliti o autorizzati ma non catturati (timeout gateway).
- Eventi duplicati o fuori ordine dalla queue RabbitMQ (idempotenza).
- Aggiornamenti concorrenti dello stesso ordine da più microservizi in parallelo.
- Fallimento totale del servizio di logistica per tempo prolungato.
- Massiccia cancellazione ordini e conseguente ripristino stock massivo.
- Rimozione o cambio di indirizzo cliente dopo conferma ordine ma prima di spedizione.
- Gestione ordini con prodotti esauriti ma non ancora scalati dal sistema.
- Scalabilità in caso di picchi di ordini stagionali o promozionali.
- Timeout o rallentamenti nelle chiamate esterne Stripe o SendGrid.
- Gestione sicurezza in caso di token JWT scaduti o compromessi.

## 5. Dipendenze e vincoli
- Dipendenza stretta tra stato ordine e workflow di pagamento e spedizione.
- Vincolo forte su transazioni ACID PostgreSQL per coerenza stock e ordini.
- Comunicazione asincrona dipende da RabbitMQ e sua corretta configurazione.
- Retry automatico presuppone idempotenza di tutte le operazioni critiche.
- Backend deve essere deployabile come microservizio indipendente senza dipendere da altri moduli.
- Sicurezza garantita da JWT per autenticazione e autorizzazione.
- Integrazione con Stripe, SendGrid e servizio logistica richiede API stable e credentials sicure.
- Audit log deve essere persistente e non modificabile per compliance e tracciabilità.
- Performance e scalabilità devono garantire esperienza utente fluida e supportare carichi variabili.

## 6. Requisiti esclusi esplicitamente
- Non è richiesto frontend, solo backend microservizio.
- Non è prevista gestione completa inventario (solo scalatura e ripristino stock per ordini).
- Non è previsto sistema di fatturazione o contabilità.
- Non è richiesto supporto a pagamenti multipli o a più gateway (solo Stripe).
- Non è richiesto tracking dettagliato della spedizione lato cliente (solo integrazione logistica).
- Nessuna menzione su supporto multilingua o localizzazione.
- Non sono richieste funzionalità di marketing o campagne promozionali.
- Non è previsto supporto per ordini di tipo non e-commerce (es. ordini telefonici).
- Non sono previsti strumenti di analisi o reporting avanzati lato backoffice.
- Non è richiesta gestione utenti cliente finale (autenticazione clienti non menzionata).