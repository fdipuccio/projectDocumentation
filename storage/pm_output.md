MODULE: PM VERSION: 1

## 1. Obiettivo
Sviluppare un microservizio backend per la gestione degli ordini e-commerce con integrazione a servizi esterni (gateway pagamento Stripe, servizio logistica, sistema notifiche SendGrid), garantendo un workflow completo dalla ricezione dell’ordine fino alla spedizione, gestione magazzino e supporto backoffice operatori.

## 2. Contesto e vincoli
- Architettura microservizio autonomo, deployabile indipendentemente in ambiente container.
- Persistenza dati su PostgreSQL con transazioni ACID per coerenza e atomicità nelle operazioni critiche.
- Comunicazione asincrona con altri microservizi tramite RabbitMQ per eventi di stato ordine.
- Integrazione con gateway pagamento Stripe (authorization e capture).
- Resilienza e affidabilità garantita tramite retry automatici su errori temporanei e operazioni idempotenti.
- Sicurezza: API protette con autenticazione e autorizzazione tramite JWT, validazione e sanificazione input.
- Logging centralizzato, audit log delle modifiche di stato ordine e operazioni critiche.
- Stack tecnologico: Java, REST API, Bear, PostgreSQL, JWT.

## 3. Assunzioni
- Payload ordine contiene dati completi: cliente, prodotti, quantità, indirizzo spedizione, dati pagamento.
- Supporto flusso completo di pagamento separato authorization + capture.
- Rimborsi manuali tramite API solo per rimborso totale; regole di business non gestite.
- Gestione magazzino con locking e controllo preventivo per evitare overselling.
- Sistema notifiche push generico configurabile (oltre a SendGrid per email), dettagli da definire.
- Sistema autenticazione utenti esterno integrato via JWT, non sviluppato internamente.
- Conservazione audit log conforme a policy GDPR con dettagli su transizioni stato, utenti, timestamp.

## 4. Scope MVP
- Implementazione API REST per ricezione ordini con validazione dati.
- Integrazione con Stripe per autorizzazione e cattura pagamenti, gestione stati pagamento.
- Workflow ordine con stati e transizioni: validazione → pagamento → conferma ordine → creazione spedizione → completamento spedizione.
- Gestione asincrona aggiornamenti stato ordine tramite consumer RabbitMQ.
- Integrazione con servizio esterno logistica per creazione spedizioni e tracking con retry e idempotenza.
- Invio notifiche via email (SendGrid) e push configurabili su ogni variazione stato ordine.
- API backoffice operatori autenticata JWT per visualizzazione lista ordini, dettaglio ordine e rimborso manuale.
- Gestione magazzino con scalatura stock a pagamento confermato e ripristino su cancellazione o rimborso.
- Logging centralizzato e audit log completo per cambi stato ordine.

## 5. Out of scope
- Frontend o interfacce utente grafiche.
- Sviluppo sistema di autenticazione utenti (si presume sistema esterno).
- Supporto a gateway di pagamento diversi da Stripe.
- Supporto multicanale o multistore.
- Sviluppo del servizio di logistica (solo integrazione).
- Supporto multi-valuta o multi-lingua.
- Reportistica avanzata o BI.
- Gestione manuale code RabbitMQ (solo consumer automatico).
- Gestione regole avanzate per rimborsi (partiali, tempistiche).

## 6. Task tecnici ordinati
1. Progettazione schema dati e tabelle PostgreSQL con transazioni ACID.
2. Sviluppo API REST per ricezione ordini (validazione payload).
3. Implementazione integrazione Stripe: authorization, capture, gestione stati, retry e idempotenza.
4. Definizione e implementazione workflow ordine con stati e transizioni.
5. Realizzazione audit log per tracciamento stato ordine e operazioni critiche.
6. Sviluppo consumer RabbitMQ per aggiornamenti asincroni stato ordine con gestione concorrenza e duplicati.
7. Integrazione con servizio esterno logistica per creazione spedizioni e tracking con meccanismi retry e idempotenza.
8. Implementazione notifica email con SendGrid e sistema push generico configurabile.
9. Creazione API backoffice protette JWT per gestione ordini (lista, dettaglio, rimborso manuale).
10. Realizzazione gestione magazzino con scalatura e ripristino stock, garantendo consistenza con locking.
11. Implementazione meccanismi di sicurezza: JWT, validazione e sanificazione input, crittografia dati sensibili.
12. Configurazione logging centralizzato, monitoraggio metriche e health check.
13. Testing funzionale, di carico, resilienza (retry, idempotenza) e sicurezza.

## 7. Acceptance criteria
- API REST ordini accettano payload validi e rigettano payload malformati.
- Integrazione Stripe funziona correttamente per authorization e capture con gestione corretta degli stati.
- Workflow ordini gestisce correttamente tutte le transizioni di stato e produce eventi asincroni.
- Consumer RabbitMQ aggiorna lo stato ordini coerentemente e non si generano duplicati o out-of-order non gestiti.
- Integrazione con logistica crea spedizioni senza duplicati e recupera tracking correttamente; gestisce errori transitori con retry.
- Notifiche email e push inviate a ogni cambio stato ordine con contenuti configurabili.
- API backoffice rispondono correttamente con autenticazione JWT, supportano filtraggio/paginazione e rimborso totale.
- Gestione magazzino aggiorna stock in modo atomico e coerente in presenza di ordini concorrenti.
- Logging e audit log tracciano tutte le operazioni critiche e transizioni con dati completi e sicurezza.
- Performance API entro 200ms per richieste standard; sistema resiste a carico di picco ordini.
- Retry automatici gestiti con backoff per servizi esterni.
- Tutte le API e servizi applicano validazione input e sanificazione per evitare vulnerabilità.

## 8. Rischi e punti aperti
- Ambiguità nel dettaglio dati ordine da ricevere via API (es. pagamenti parziali o multipli).
- Gestione dettagliata dei rimborsi manuali (parziale, totale, tempistiche) da definire.
- Dettagli sistema notifiche push e canali supportati ancora da chiarire.
- Possibili scenari di concorrenza su magazzino in ordini simultanei con stock limitato.
- Gestione messaggi duplicati o fuori ordine da RabbitMQ.
- Impatto e gestione degradata se servizio logistica esterno è indisponibile a lungo.
- Integrazione con sistema autenticazione utenti esterno deve essere confermata.
- Nessun supporto multi-tenant/multi-store previsto al momento, ma possibile evoluzione futura.
- Policy di conservazione audit log perfettamente allineate con GDPR da verificare.

---

Documento elaborato con attenzione a distinguere chiaramente assunzioni da vincoli tecnici e funzionali confermati. Non sono stati aggiunti requisiti esterni rispetto a quelli forniti.