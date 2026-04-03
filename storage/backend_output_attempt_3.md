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