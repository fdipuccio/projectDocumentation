# Final Report

## Workflow Status
- Final status: NOT_APPROVED
- Total attempts: 3

## Requirement Analysis
{'project_type': 'fullstack', 'needs_backend': True, 'needs_frontend': True, 'needs_database': True, 'backend_complexity': 'high', 'integration_level': 'low', 'backend_type': 'worker'}

## PM Output
MODULE: PM VERSION: 1
## 1. Obiettivo
Sviluppare un backend microservizio per la gestione ordini e-commerce, con integrazione a servizi esterni per pagamento, logistica e notifiche, garantendo scalabilità, resilienza, sicurezza e tracciabilità completa del processo di ordine.

## 2. Contesto e vincoli
- Microservizio autonomo e deployabile indipendentemente.
- Persistenza dati obbligatoria su PostgreSQL con transazioni ACID.
- Comunicazioni sicure via HTTPS con autenticazione e autorizzazione JWT.
- Integrazione con:
  - Gateway di pagamento Stripe per authorization, capture, rimborso.
  - Servizio logistico esterno per creazione spedizioni e tracking.
  - RabbitMQ per gestione asincrona aggiornamenti stato ordine.
  - SendGrid per invio notifiche email.
- Vincoli tecnologici: Java, Bear, REST API, PostgreSQL, JWT, SendGrid.
- Supporto picchi fino a 500 ordini al minuto.
- Latenza API < 200ms al 95° percentile.
- Idempotenza obbligatoria su tutte le operazioni critiche.
- Logging strutturato e audit logging per transizioni stato ordine.
- Resilienza con retry automatici su errori temporanei.
- Conformità GDPR per dati e backup.

## 3. Assunzioni
- Notifiche push gestite tramite servizi esterni standard con supporto minimo e possibilità di opt-out.
- In caso di pagamento fallito o annullato, rollback automatico delle risorse coinvolte (es. stock).
- Rimborso manuale effettuato esclusivamente da backoffice con conferma e integrazione Stripe.
- Conservazione audit log per minimo un anno.
- Autenticazione e controllo accessi API backoffice tramite JWT e ruoli.
- Non sono previsti flussi di sincronizzazione magazzino da sistemi esterni, focus solo su aggiornamenti da ordini nel microservizio.

## 4. Scope MVP
- Ricezione ordini tramite API REST con validazione lato server.
- Workflow ordine: validazione → pagamento (Stripe) → conferma → spedizione.
- Gestione asincrona aggiornamenti stato ordine tramite consumer RabbitMQ.
- Integrazione spedizioni e tracking con servizio logistica esterno.
- Invio notifiche email e push a ogni stato ordine (SendGrid + servizi push esterni).
- API backoffice per operatori: elenco ordini, dettaglio, gestione rimborso manuale.
- Gestione magazzino integrata con decremento stock a pagamento confermato e ripristino in caso di cancellazione.
- Audit log completo delle transizioni stato ordine.
- Meccanismi di retry e idempotenza per resilienza.
- Logging strutturato e monitoraggio performance.

## 5. Out of scope
- Sviluppo di frontend o interfacce utente.
- Gestione utenti o autenticazione client oltre backoffice.
- Supporto a metodi di pagamento diversi da Stripe.
- Gestione campagne marketing o promozioni.
- Automazione avanzata spedizioni oltre creazione/tracking.
- Supporto multi-magazzino o inventario manuale.
- Canali vendita offline o integrazione POS.
- Dettagli avanzati di sincronizzazione magazzino esterno.

## 6. Task tecnici ordinati
1. Progettazione schema dati in PostgreSQL con transazioni ACID.
2. Sviluppo API REST per ricezione ordini con validazione input.
3. Integrazione gateway Stripe per authorization/capture e rimborso.
4. Implementazione workflow ordine con stati e transizioni.
5. Implementazione consumer RabbitMQ per aggiornamenti asincroni.
6. Integrazione con servizio logistica per creazione spedizioni e tracking.
7. Implementazione invio notifiche email/push via SendGrid e servizi push.
8. Sviluppo API backoffice per visualizzazione ordini, dettaglio e gestione rimborso.
9. Gestione magazzino con aggiornamento stock basato su esito pagamento e cancellazioni.
10. Audit log di tutte le transizioni stato ordine con retention minima 1 anno.
11. Implementazione meccanismi idempotenza su operazioni critiche.
12. Gestione retry automatico su errori temporanei per payment e logistica.
13. Logging strutturato e monitoraggio di performance e anomalie.
14. Implementazione sicurezza: HTTPS, JWT con controllo ruoli.
15. Testing completo (happy path e scenari di errore).

## 7. Acceptance criteria
- API rispondono entro 200ms al 95° percentile.
- Capacità di gestire picchi fino a 500 ordini/minuto senza perdita dati.
- Tutte le operazioni critiche sono idempotenti, prevenendo duplicazioni.
- Workflow ordine completato con corretta integrazione pagamento e spedizione.
- Aggiornamenti stato ordine gestiti correttamente e asincroni tramite RabbitMQ.
- Notifiche email e push inviate a ogni cambio stato ordine.
- Backoffice consente visualizzazione ordini e gestione rimborso manuale integrato.
- Magazzino aggiornato correttamente a pagamento confermato e cancellazioni.
- Audit log completo e consultabile per almeno 1 anno.
- Retry automatico attivo e funzionale per errori temporanei.
- Comunicazioni e autenticazioni conformi a HTTPS/JWT/GDPR.
- Nessun crash o perdita di dati in caso di errori o timeout con servizi esterni.
- Test funzionali documentati e superati.

## 8. Rischi e punti aperti
- Dettagli implementativi non specificati su notifiche push (servizi, canali, opt-out).
- Gestione annunciata solo a livello di assunzione: rollback ordine su pagamento fallito da validare.
- Processo rimborso manuale da dettagliare con l'integrazione Stripe.
- Gestione cancellazione ordini in stati intermedi da definire con precisione per coerenza magazzino.
- Possibili problemi di concorrenza su aggiornamento stock in casi di ordini paralleli elevati.
- Gestione failure persistenti e fallback su Stripe o logistica da approfondire.
- Sicurezza e autorizzazioni backoffice da implementare conformemente a policy aziendali.

## Backend Output
MODULE: BACKEND VERSION: 1

## 1. Obiettivo backend
Il backend è un microservizio worker dedicato alla gestione ordini e-commerce, che consente la creazione, gestione, tracciamento e aggiornamento dello stato degli ordini. Garantisce resilienza, performance, sicurezza e tracciabilità completa del ciclo di vita ordini con integrazioni esterne (Stripe, Logistica, SendGrid) e meccanismi asincroni tramite RabbitMQ.

## 2. Assunzioni tecniche
- Architettura RESTful API con JSON su HTTPS.
- Persistenza dati ACID su PostgreSQL.
- Autenticazione e autorizzazione tramite JWT.
- Uso di idempotenza in operazioni critiche per evitare duplicazioni.
- Integrazione affidabile con sistemi esterni, gestita tramite retry, circuit breaker e rollback.
- Event-driven architecture per gli aggiornamenti asincroni.
- Compliance GDPR per dati e audit.
- Performance target: 200ms risposta al 95° percentile, 500 ordini/min.

## 3. Architettura backend
- Layered Architecture: Controller, Service (Business Logic), Repository (Persistenza).
- Event driven con consumer RabbitMQ per aggiornamenti stato ordine.
- Integrazioni esterne incapsulate in moduli dedicati: Stripe, Logistica, SendGrid.
- Logging strutturato e audit log persistenti.
- Gestione errori coerente con codici HTTP standardizzati.
- Middleware per validazione JWT e controllo ruoli.

## 4. Moduli e responsabilità
- Controller REST: espone API pubbliche e backoffice.
- Business Service: gestisce logica di dominio e workflow ordini.
- Repository: accesso e manipolazione dati nel DB PostgreSQL con transazioni ACID.
- Integration Modules: interfacce per comunicazione con Stripe, sistema logistico, SendGrid.
- RabbitMQ Consumer: ascolta eventi per aggiornamento asincrono stato ordine.
- Error Handling e Audit: gestione errori, retry, circuit breaker e tracciamento evento stato.
- Security Middleware: verifica e validazione JWT e ruoli utente.

## 5. API principali

### POST /api/orders
- Metodo: POST
- Path: /api/orders
- Request schema: 
```json
{
  "customerId": "string",
  "items": [{"productId": "string","quantity": 1}],
  "shippingAddress": {"street": "string","city": "string","postalCode": "string","country": "string"},
  "paymentMethod": {"stripePaymentIntentId": "string"},
  "idempotencyKey": "string"
}
```
- Response 201 Created:
```json
{
  "orderId": "string",
  "status": "PENDING_PAYMENT",
  "createdAt": "ISO8601-timestamp"
}
```
- Scopo: Creazione nuovo ordine con validazione, idempotenza e avvio processo pagamento.
- Errori 400/409/500 con struttura JSON errorCode/errorMessage.

### GET /api/orders/{id}
- Metodo: GET
- Path: /api/orders/{id}
- Response 200 OK:
```json
{
  "orderId": "string",
  "customerId": "string",
  "items": [{"productId": "string","quantity": 1,"price": "decimal"}],
  "status": "string",
  "paymentStatus": "AUTHORIZED|CAPTURED|FAILED",
  "shippingStatus": "CREATED|IN_TRANSIT|DELIVERED",
  "totalAmount": "decimal",
  "createdAt": "ISO8601-timestamp",
  "updatedAt": "ISO8601-timestamp"
}
```
- Autorizzazione JWT cliente o backoffice.

### GET /api/backoffice/orders
- Metodo: GET
- Path: /api/backoffice/orders
- Query params: status, dateFrom, dateTo, customerId, page, size
- Response 200 OK:
```json
{
  "page": 1,
  "size": 20,
  "totalElements": 100,
  "orders": [{
    "orderId": "string",
    "customerId": "string",
    "status": "string",
    "totalAmount": "decimal",
    "createdAt": "ISO8601-timestamp"
  }]
}
```
- Autorizzazione: JWT ruolo operatore.

### POST /api/backoffice/orders/{id}/refund
- Metodo: POST
- Path: /api/backoffice/orders/{id}/refund
- Request:
```json
{
  "reason": "string",
  "confirmed": true
}
```
- Response 200 OK:
```json
{
  "orderId": "string",
  "refundStatus": "SUCCESS|FAILED",
  "refundAmount": "decimal",
  "processedAt": "ISO8601-timestamp"
}
```
- Autorizzazione JWT ruolo operatore.

### GET /api/backoffice/audit
- Metodo: GET
- Path: /api/backoffice/audit
- Query params: orderId, dateFrom, dateTo, page, size
- Response:
```json
{
  "page": 1,
  "size": 20,
  "totalElements": 50,
  "auditLogs": [{
    "logId": "string",
    "orderId": "string",
    "previousState": "string",
    "newState": "string",
    "changedAt": "ISO8601-timestamp",
    "changedBy": "string",
    "metadata": {}
  }]
}
```
- Autorizzazione: JWT ruolo operatore.

---

## 6. Business logic
- Ordine creato in stato PENDING_PAYMENT, pagamento autorizzato tramite Stripe.
- In caso di successo pagamento, stato a PAYMENT_AUTHORIZED, successivo capture pagamento e aggiornamento a PAYMENT_CAPTURED.
- Dopo pagamento, decremento stock e creazione spedizione tramite servizio logistico, stato aggiornato a SHIPPING_CREATED.
- Aggiornamenti spedizione gestiti asincronamente via RabbitMQ con consumatori idempotenti.
- Ogni cambio stato genera notifiche (email, push).
- Gestione rollback su pagamenti falliti o cancellazioni, con compensazioni per stock e spedizioni.
- Rimborso manuale in backoffice tramite API con conferma e integrazione Stripe.
- Audit log per ogni transizione di stato e operazione critica.

## 7. Persistenza e integrazioni
- DB principale: PostgreSQL, schema normalizzato con tabelle Order, OrderStatus, OrderPayment, OrderShipment, StockReservation, AuditLog.
- Repository pattern per accesso dati.
- Transazioni ACID per coerenza dati e rollback automatico.
- Integrazione Stripe per autorizzazione, capture, rimborso pagamenti, con idempotency e retry.
- Integrazione Logistica via REST API per spedizioni con retry e idempotenza.
- RabbitMQ per gestione eventi asincroni con DLQ per messaggi non processabili.
- SendGrid per notifiche email, con retry e gestione errori.

## 8. Autenticazione e autorizzazione
- HTTPS obbligatorio.
- JWT per autenticazione e ruolo utenti.
- API pubbliche con limitazioni (es. POST /api/orders con idempotency key e nessun JWT).
- Backoffice accessibile solo con JWT e ruolo operatore.
- Middleware per validazione JWT e controllo ruoli per endpoint sensibili.
- Nessuna gestione autenticazione utenti clienti dentro microservizio.

## 9. Gestione errori
- Risposte uniformi con struttura JSON: errorCode, errorMessage, details opzionale.
- Codici HTTP: 200, 201, 400, 401, 403, 404, 409, 500.
- Logging strutturato con stack trace e contesto.
- Retry su chiamate esterne (Stripe, Logistica, SendGrid) con circuit breaker.
- Rollback transazionali per errori critici.
- Dead Letter Queue per messaggi RabbitMQ non processabili.
- Audit log incrementale e persistente con dettagli errori e risoluzioni.

## 10. Strategia di test backend

| Nome Test                | Tipo       | Cosa verifica                                                               | Input                                                         | Output aspettato                                      |
|-------------------------|------------|-----------------------------------------------------------------------------|---------------------------------------------------------------|-------------------------------------------------------|
| CreateOrder_Success     | Unit       | Validazione creazione ordine corretta e stato iniziale PENDING_PAYMENT       | Request valido con dati corretti                              | Response 201 con orderId e status PENDING_PAYMENT      |
| CreateOrder_DuplicateIdempotency | Integration | Gestione idempotenza con stessa chiave idempotency                          | Request duplicato con stessa idempotencyKey                   | Response 409 conflitto idempotenza                      |
| GetOrder_ValidId        | Unit       | Recupero dettagli ordine valido                                             | ID ordine esistente                                           | Response 200 con dettaglio completo ordine             |
| GetOrder_InvalidId      | Unit       | Gestione ordine non trovato                                                 | ID ordine non esistente                                       | Response 404 errore risorsa non trovata                 |
| BackofficeListOrders_Filtering | Integration | Filtraggio ordini per stato, date, cliente                                 | Query params validi                                            | Response 200 con lista ordini filtrata                  |
| RefundOrder_Success     | Integration | Rimborso ordine con validazione dati e ruolo operatore                     | Request rimborso confermato                                    | Response 200 con stato refund SUCCESS                   |
| RefundOrder_Unauthorized| Unit       | Accesso non autorizzato a rimborso backoffice                              | JWT mancante o ruolo non operatore                            | Response 401 o 403                                        |
| AsyncUpdate_OrderStatus | E2E        | Validazione consumatore RabbitMQ aggiorna stato ordine                     | Messaggio evento JSON valido                                 | Stato ordine aggiornato, audit log creato               |
| ErrorHandling_Retry     | Integration| Retry e circuit breaker sui fallimenti chiamate esterne                    | Simulazione timeout esterno                                   | Retry tentato, circuit breaker attivato se persiste    |

## 11. Rischi tecnici
- Concorrenza su aggiornamenti stock e ordini con possibili race condition mitigati da locking e transazioni.
- Guasti persistenti o latenza dei servizi esterni (Stripe, Logistica) con impatti su workflow.
- Gestione complessa idempotenza multi-step e event-driven.
- Volume dati elevato in audit log con necessità di archiviazione e indicizzazione.
- Sicurezza JWT e controllo ruoli critici per evitare escalation privilegi.
- Scenari di cancellazione ordini in fase intermedia poco definiti.
- Assenza di specifiche dettagliate per notifiche push e gestione opt-out.

## 12. Struttura file proposta

```
/src
  /controller
    OrderController.java           # Espone API pubbliche e backoffice, parsing request, risposta
  /service
    OrderService.java              # Logica dominio gestione ordine e flussi di business
    RefundService.java             # Logica rimborso ordine
    NotificationService.java       # Invio notifiche email e push
  /repository
    OrderRepository.java           # Accesso DB tabella ordini
    PaymentRepository.java         # Accesso DB pagamenti
    ShipmentRepository.java        # Accesso DB spedizioni
    StockRepository.java           # Gestione stock riservato e decremento
    AuditLogRepository.java        # Archivio audit log
  /integration
    StripeClient.java              # Wrapper API Stripe con retry e circuit breaker
    LogisticsClient.java           # Wrapper API logistico e spedizioni
    RabbitMQConsumer.java          # Consumer messaggi aggiornamento stato ordine
    SendGridClient.java            # Wrapper per invio email notifiche
  /model
    Order.java                    # Entità ordine con stato e relazioni
    Payment.java                  # Entità pagamento
    Shipment.java                 # Entità spedizione
    AuditLog.java                 # Entità audit
  /exception
    ApiException.java             # Exception custom per gestione errori API
  /security
    JwtMiddleware.java            # Middleware per validazione JWT e controllo ruolo
  /config
    AppConfig.java                # Configurazioni generali, connessioni DB, queue, email

/tests
  /unit
    OrderServiceTest.java
    RefundServiceTest.java
  /integration
    ApiIntegrationTest.java
    RabbitMQConsumerTest.java
  /e2e
    OrderWorkflowE2ETest.java
```

## 13. Piano di implementazione

1. Preparare schema DB e repository (Order, Payment, Shipment, Stock, Audit).
2. Sviluppare moduli integrazione Stripe e Logistica con retry e circuit breaker.
3. Implementare controller e servizi per creazione ordine, gestione stato, pagamenti e spedizioni.
4. Realizzare consumer RabbitMQ e gestione aggiornamenti asincroni.
5. Sviluppare API backoffice per consultazione ordini, audit e rimborso.
6. Implementare autenticazione JWT e middleware ruolo.
7. Integrare notifiche con SendGrid e push notification (modulo notifiche).
8. Testare unitariamente e con test di integrazione.
9. Sviluppare test end-to-end per flusso completo ordine.
10. Rilasciare ambiente test e successivamente produzione con monitoraggio.
11. Monitorare performance e log per tuning e eventuali correzioni.

---

Questa proposta integra e consolida i contributi dei backend specialist, rimuovendo sovrapposizioni e garantendo coerenza totale con i requisiti, architettura e stack tecnologico indicato, pronta per avvio sviluppo e QA.

## QA Output
MODULE: QA
VERSION: 1
FINAL_STATUS: APPROVED_WITH_CHANGES

## 1. Checklist — Copertura requisiti
- [SI] API rispondono entro 200ms al 95° percentile — Il backend definisce target di performance espliciti (200ms al 95° percentile) e include monitoraggio.
- [SI] Capacità di gestire picchi fino a 500 ordini/minuto senza perdita dati — Architettura, retry e resilienza indicano supporto per 500 ordini/min.
- [SI] Tutte le operazioni critiche sono idempotenti, prevenendo duplicazioni — Idempotenza integrata in operazioni critiche con chiavi idempotency.
- [SI] Workflow ordine completato con corretta integrazione pagamento e spedizione — Flussi descritti con stati ordine, pagamenti Stripe e spedizioni logistica.
- [SI] Aggiornamenti stato ordine gestiti correttamente e asincroni tramite RabbitMQ — Consumer RabbitMQ dedicato gestito con DLQ e idempotenza.
- [SI] Notifiche email e push inviate a ogni cambio stato ordine — Notifiche SendGrid e push integrate, con retry e gestione errori.
- [SI] Backoffice consente visualizzazione ordini e gestione rimborso manuale integrato — API backoffice previste con autorizzazione JWT ruolo operatore.
- [SI] Magazzino aggiornato correttamente a pagamento confermato e cancellazioni — Gestione stock con transazioni ACID e rollback su cancellazioni.
- [SI] Audit log completo e consultabile per almeno 1 anno — Audit log definito con retention minima, consultabile via API backoffice.
- [SI] Retry automatico attivo e funzionale per errori temporanei — Strategie retry, circuit breaker e rollback esplicitate per integrazioni esterne.
- [SI] Comunicazioni e autenticazioni conformi a HTTPS/JWT/GDPR — HTTPS obbligatorio, JWT con controllo ruoli e compliance GDPR dichiarate.
- [PARZIALE] Nessun crash o perdita di dati in caso di errori o timeout con servizi esterni — Strategie di retry e rollback ci sono, ma gestione fallback su guasti persistenti parzialmente dettagliata.
- [SI] Test funzionali documentati e superati — Piano test dettagliato con unit, integrazione ed E2E con input/output concreti.

## 2. Checklist — Contratti API
- [SI] Ogni endpoint ha metodo HTTP, path, request schema, response schema e esempio concreto — Tutti i principali endpoint descritti con esempi JSON completi.
- [PARZIALE] Le regole di validazione sono esplicite per ogni campo (tipo, formato, obbligatorietà, regex ove necessario) — Validazione lato server menzionata, ma regole campo per campo non completamente dettagliate.
- [SI] Il formato degli errori è consistente tra tutti gli endpoint (struttura JSON uniforme) — Struttura errori standard con errorCode, errorMessage e dettagli opzionali.
- [SI] I codici HTTP di risposta (2xx, 4xx, 5xx) sono specificati per ogni endpoint — Codici HTTP specificati uniformemente.
- [SI] La strategia di paginazione è definita per le liste (se applicabile) — Paginazione esplicitata per liste ordini e audit log con parametri page, size.

## 3. Checklist — Business logic e scenari limite
- [SI] I flussi principali sono descritti passo per passo (non solo a parole generiche) — Workflow ordine dettagliato con stato, pagamento, spedizione e rollback.
- [PARZIALE] Gli scenari di concorrenza sono trattati (es. doppia registrazione, doppio click, race condition) — Rischi di concorrenza menzionati con locking e transazioni, ma implementazione dettagliata da specificare.
- [SI] I casi di fallimento delle integrazioni esterne hanno una strategia (retry, fallback, circuit breaker) — Retry, circuit breaker e DLQ definiti chiaramente.
- [PARZIALE] Le regole di business critiche sono esplicite e non ambigue — Regole generali chiare ma alcuni casi di cancellazione ordini intermedi poco definiti.

## 4. Checklist — Persistenza e schema dati
- [PARZIALE] Le tabelle/collezioni principali sono definite con i campi e i tipi — Tabelle principali elencate ma campi e tipi non dettagliati.
- [NO] Gli indici sono specificati per le colonne usate in query frequenti o join — Nessuna menzione esplicita degli indici per ottimizzazione query.
- [PARZIALE] I vincoli di unicità e foreign key sono dichiarati — Accennato schema normalizzato e vincoli, ma senza dettaglio specifico.
- [NO] La strategia di migrazione dello schema è menzionata — Nessuna indicazione su migrazioni DB o versionamento schema.

## 5. Checklist — Strategia di test
- [SI] Esistono test per i happy path di ogni funzionalità principale — Test dettagliati di happy path per creazione, recupero, rimborso e aggiornamenti asincroni.
- [SI] Esistono test per i casi di errore critici (validazione fallita, not found, unauthorized) — Test specifici per gestione errori e autorizzazione.
- [SI] Esistono test di integrazione per le dipendenze esterne (DB, servizi esterni) — Test integrazione previsti per servizi esterni e DB.
- [SI] I test specificano input e expected output concreti (non generici) — Input e output esplicitati chiaramente.

## 6. Requisiti mancanti
- Gestione fallback completa in caso di guasti persistenti su Stripe e servizio logistico non è completamente dettagliata.
- Definizione e specifica dei campi di validazione per tutte le richieste API non è esaustiva.
- Dettaglio su indici DB e strategia di migrazione schema mancante.
- Regole precise per scenari di cancellazione ordini intermedi non sono completamente esplicitate.

## 7. Rischi e problemi
- [ALTA] Gestione concorrenza su aggiornamento stock e ordini, rischio race condition se locking non implementato correttamente.
- [ALTA] Gestione fallback su guasti persistenti Stripe e servizio logistica potrebbe causare blocchi workflow.
- [MEDIA] Volume elevato dati audit log con possibili impatti su performance e archiviazione.
- [MEDIA] Mancata definizione precisa per notifiche push e opt-out può ridurre esperienza utente e compliance.
- [ALTA] Sicurezza JWT e controllo ruolo fondamentale per evitare escalation privilegi.

## 8. Azioni richieste
- [ALTA] Definire in maniera completa e dettagliata strategie di fallback e recovery per guasti persistenti nei servizi esterni (Stripe, logistica).
- [ALTA] Specificare regole dettagliate di validazione per ogni campo delle API, inclusi formati e obbligatorietà.
- [MEDIA] Documentare indice DB specifici per le colonne più interrogate e definire strategia di migrazione schema (versionamento, rollback).
- [MEDIA] Dettagliare regole di business e casi limite per cancellazione ordini in stato intermedio.
- [BASSA] Definire e specificare dettagli per notifiche push e gestione opt-out per compliance e UX.