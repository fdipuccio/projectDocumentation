MODULE: PM VERSION: 1
## 1. Obiettivo
Sviluppare un backend microservizio per la gestione ordini e-commerce, che includa la ricezione ordini tramite API REST, integrazione con gateway di pagamento Stripe per authorization e capture, workflow di elaborazione ordine, consumo asincrono di eventi da RabbitMQ, integrazione con servizi di logistica esterna, invio di notifiche email/push tramite SendGrid, API di backoffice per operatori per gestione ordini e rimborsi manuali, e gestione magazzino per scalatura e ripristino stock. Il sistema deve garantire sicurezza, resilienza, tracciabilità completa e idempotenza delle operazioni critiche.

## 2. Contesto e vincoli
- Architettura microservizio autonomo e deployabile indipendentemente.
- Persistenza dati su PostgreSQL con transazioni ACID obbligatorie per operazioni critiche.
- Comunicazione asincrona attraverso RabbitMQ.
- Integrazione con Stripe per pagamenti (authorization, capture, refund) con idempotenza.
- Invio notifiche tramite SendGrid e servizi push esterni.
- Sicurezza tramite autenticazione e autorizzazione JWT, comunicazioni via HTTPS.
- Audit log di tutte le transizioni di stato ordine, conservato per almeno un anno con natura immutabile.
- Gestione retry automatico per errori transitori su servizi di pagamento e logistica.
- Stack tecnologico: Java, REST API, Bear, PostgreSQL, JWT.

## 3. Assunzioni
- Il flusso di rimborso manuale gestisce sia il rimborso importo pagamento sia il ripristino stock.
- Il servizio esterno di logistica, in caso di indisponibilità, sarà gestito con retry automatici.
- Le notifiche push sono generiche e fornite tramite SendGrid o servizi esterni con supporto opt-out.
- La gestione concorrente della scalatura stock sarà basata su transazioni ACID di PostgreSQL per garantire coerenza.
- L'audit log comprende un dettaglio esaustivo delle transizioni di stato ordine ma non necessariamente di tutti i dati modificati.
- L’API backoffice è destinata esclusivamente a operatori interni aziendali.
- Non è prevista gestione di ordini parzialmente spediti o rimborsati per la MVP, da valutare in futuri sviluppi.

## 4. Scope MVP
- Endpoint API REST per creazione e ricezione ordini con validazione input backend.
- Integrazione piena con Stripe per pagamento (authorization e capture).
- Workflow ordine completo con stati e registrazione delle transizioni in audit log.
- Consumer RabbitMQ per eventi di aggiornamento stato ordine, con elaborazione idempotente.
- Integrazione base con servizio logistica per creazione spedizioni e aggiornamenti tracking con retry.
- Invio notifiche email e push via SendGrid ad ogni cambio stato ordine.
- API backoffice per operatori: lista ordini, dettaglio ordine e rimborso manuale completo di stock.
- Gestione stock: decremento al pagamento confermato e ripristino su cancellazione o rimborso.
- Gestione sicurezza tramite JWT per autenticazione e autorizzazione API.
- Logging strutturato e audit compliance minima 1 anno.

## 5. Out of scope
- Frontend o interfacce utente diverse dalle API backend.
- Gestione completa inventario oltre scalatura e ripristino stock per ordini.
- Sistemi di fatturazione o contabilità.
- Supporto a più gateway di pagamento oltre Stripe.
- Tracking spedizione dettagliato lato cliente.
- Supporto multilingua o localizzazione.
- Funzionalità marketing, campagne promozionali o ordini non e-commerce.
- Gestione utenti cliente finale o autenticazione clienti.
- Reporting analitico o strumenti di analisi avanzata backoffice.
- Gestione ordini parzialmente spediti o rimborsati nella MVP.

## 6. Task tecnici ordinati
1. Definizione modello dati ordini, pagamenti, stock, audit log conforme ACID.
2. Sviluppo API REST per ricezione ordini con validazione backend.
3. Implementazione integrazione Stripe per authorization, capture e rimborso.
4. Realizzazione workflow ordine come state machine con logging transizioni.
5. Sviluppo consumer RabbitMQ per eventi aggiornamento stato ordine con idempotenza.
6. Integrazione con servizio esterno logistica per creazione spedizioni e tracking, con retry.
7. Implementazione invio notifiche email e push tramite SendGrid.
8. Sviluppo API backoffice per gestione ordini: listaggio, dettaglio e rimborso manuale.
9. Implementazione logica gestione magazzino: scalatura e ripristino stock.
10. Applicazione sicurezza JWT su tutte le API con validazione token e gestione errori.
11. Predisposizione audit log persistente, immutabile e compliance GDPR.
12. Realizzazione sistema retry automatico per integrazioni esterne.
13. Testing unitario, integrazione e di sicurezza.
14. Deployment microservizio e configurazione monitoring e alerting.

## 7. Acceptance criteria
- API REST ordini che accettano e validano input corretti e rifiutano invalidi.
- Pagamenti Stripe autorizzati, catturati e rimborsati con successo, con idempotenza garantita.
- Workflow ordine attivo con stati e transizioni tracciate su audit log.
- Eventi RabbitMQ processati in modo affidabile, senza duplicazioni.
- Servizio logistica integrato con retry configurati e gestione errori.
- Notifiche email e push inviate correttamente ad ogni stato ordine.
- API backoffice funzionante per listare ordini, visualizzarne dettagli e gestire rimborsi manuali.
- Stock aggiornato coerentemente con pagamenti e cancellazioni.
- Autenticazione e autorizzazione JWT attiva e funzionante per tutte le API protette.
- Audit log persistente almeno per un anno, immutabile e consultabile.
- Microservizio deployato, monitorato e performante sotto carico previsto.
- Test completati con copertura soddisfacente e errori gestiti correttamente.

## 8. Rischi e punti aperti
- Gestione ordini parzialmente spediti o rimborsati non specificata e rimandata.
- Possibili picchi elevati di richieste con rischio di rallentamenti o locking database.
- Coordinamento con servizi esterni (Stripe, logistica, SendGrid) e gestione dei loro downtime.
- Eventuali limitazioni o scadenze token JWT potrebbero impattare usabilità API.
- Dettaglio delle notifiche push e loro integrazione esterna può richiedere ulteriori chiarimenti.
- Necessità di definire politiche di retention e backup audit log per compliance.
- Potenziali race condition concorrenti su aggiornamenti stock in scenari di alta concorrenza.
- Adozione delle corrette strategie di retry e fallback per resilienza.
- Privacy e sicurezza dati sensibili lato pagamenti e clienti devono essere garantite secondo GDPR.