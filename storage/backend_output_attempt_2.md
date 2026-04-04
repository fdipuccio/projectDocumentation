MODULE: BACKEND VERSION: 1

## 1. Obiettivo backend
Sviluppare un microservizio backend di tipo worker dedicato alla gestione ordini di un e-commerce. Il sistema deve orchestrare in modo sicuro e scalabile le operazioni di creazione, validazione, pagamento, gestione stock e spedizione degli ordini, garantendo compliance GDPR e PCI-DSS, bassa latenza, idempotenza nelle operazioni critiche, tracciabilità via audit log e integrazioni robuste con sistemi esterni (pagamenti Stripe, logistica, notifiche).

## 2. Assunzioni tecniche
- Architettura basata su microservizi con comunicazione sincrona REST e asincrona event-driven tramite RabbitMQ.
- Uso di PostgreSQL come singola fonte di verità con transazioni ACID.
- Tutte le operazioni critiche (creazione ordine, chiamate pagamento, logistica) idempotenti via token univoci.
- Autenticazione e autorizzazione via JWT Bearer Token esterni, con verifica ruoli per backoffice.
- Gestione errori centralizzata con codici errori standard, retry automatici e dead-letter per eventi.
- Performance garantite con risposte <200 ms al 95° percentile.
- Conformità GDPR e PCI-DSS per dati sensibili.
- Modello di dominio coerente con gestione stadi ordine, pagamenti, inventario, spedizione e audit immutabile.

## 3. Architettura backend
- Microservizi strutturati in moduli distinti: API ordini, pagamento (Stripe), logistica, notifiche, e audit logging.
- API REST sincrone per operazioni CRUD e backoffice.
- Gestione eventi asincroni con consumer RabbitMQ per aggiornamenti stato ordine con deduplicazione e ordering.
- Persistenza tramite repository con uso di ORM (JPA o simile), ottimizzato per carichi elevati e concorrenza.
- Gestione idempotenza e concorrenza tramite locking ottimistico e token idempotenza.
- Logging strutturato per audit e monitoraggio.
- Retry esponenziali per integrazioni esterne critiche.

## 4. Moduli e responsabilità
- Order API Module: espone endpoint CRUD ordine, validazione input, gestione stato.
- Payment Module (Stripe integration): autorizzazione e cattura pagamenti, gestione idempotenza e retry.
- Logistics Integration Module: creazione spedizioni, tracking, sincronizzazione stato.
- Notification Module: invio email via SendGrid con retry.
- Audit Logging Module: persistenza immutabile di eventi di stato e operazioni critiche.
- RabbitMQ Consumer Module: gestione asincrona eventi ordini, processamento idempotente e ordinato.
- Repository Module: astrazione DB con pattern repository per ogni entità.
- Security Module: gestione autenticazione JWT, autorizzazione role-based, sanitizzazione input.

## 5. API principali

### 5.1 Create Order
- Metodo: POST
- Path: /api/orders
- Request Schema:
```json
{
  "customerId": "string",
  "items": [{ "productId": "string", "quantity": integer }],
  "paymentMethod": { "type": "card", "stripePaymentMethodId": "string" },
  "shippingAddress": {
    "line1": "string", "line2": "string", "city": "string",
    "postalCode": "string", "country": "string"
  },
  "metadata": { "notes": "string" }
}
```
- Headers: Authorization (Bearer JWT), Idempotency-Key (string unico)
- Response Schema (201 Created):
```json
{
  "orderId": "string",
  "status": "VALIDATION_PENDING",
  "createdAt": "ISO8601 timestamp"
}
```
- Esempio:
```http
POST /api/orders
Headers:
Authorization: Bearer eyJ...
Idempotency-Key: abc123xyz
Body:
{
  "customerId": "cust01",
  "items": [{ "productId": "prod123", "quantity": 2 }],
  "paymentMethod": { "type": "card", "stripePaymentMethodId": "pm_1" },
  "shippingAddress": {
    "line1": "Via Roma 1", "city": "Milano",
    "postalCode": "20100", "country": "IT"
  },
  "metadata": { "notes": "Regalo" }
}
Response:
{
  "orderId": "ord0001",
  "status": "VALIDATION_PENDING",
  "createdAt": "2024-06-01T12:00:00Z"
}
```

### 5.2 Get Order Detail
- Metodo: GET
- Path: /api/orders/{orderId}
- Response Schema:
```json
{
  "orderId": "string",
  "status": "VALIDATION_PENDING | PAYMENT_AUTHORIZED | PAYMENT_CAPTURED | CONFIRMED | SHIPPED | CANCELLED",
  "items": [{ "productId": "string", "quantity": integer }],
  "payment": {
    "authorizationId": "string",
    "captured": boolean,
    "amount": number,
    "currency": "string"
  },
  "shipping": {
    "address": { /* stesso schema shippingAddress */ },
    "shipmentId": "string",
    "trackingUrl": "string"
  },
  "auditTrail": [
    { "event": "string", "timestamp": "ISO8601", "actor": "system|operator|external", "details": {} }
  ]
}
```

### 5.3 List Orders (Backoffice)
- Metodo: GET
- Path: /api/backoffice/orders
- Query params: filter stato, range date, paginazione
- Accesso: operatori JWT con ruolo minimo
- Risposta: lista ordini con sintesi stati e date

### 5.4 Issue Refund (Backoffice)
- Metodo: POST
- Path: /api/backoffice/orders/{orderId}/refund
- Request Schema:
```json
{
  "reason": "string"
}
```
- Accesso: operatori JWT con ruolo admin
- Idempotenza: token header
- Risposta: 200 OK con stato rimborso

## 6. Business logic
- Ciclo vita ordine gestito in fasi: VALIDATION_PENDING → PAYMENT_AUTHORIZED → PAYMENT_CAPTURED → CONFIRMED → SHIPPED (o CANCELLED).
- Validazione ordini: controllo stock con lock ottimistico tramite Inventory Module.
- Gestione pagamenti: autorizzazione e cattura distinte su Stripe, retry sicuro, rollback stock su errori o cancellazioni.
- Invio spedizioni tramite Logistics Module con idempotenza e aggiornamento asincrono stato via RabbitMQ.
- Notifiche email su ogni cambiamento stato, retry gestito dal Notification Module.
- Backoffice per gestione ordini e rimborsi con auditing rigoroso.
- Event handling asincrono con deduplicazione e ordinamento eventi per garantire consistenza.
- Log immutabile di tutte le transizioni e operazioni per audit e compliance.

## 7. Persistenza e integrazioni
- Database: PostgreSQL con schema relazionale, chiavi esterne, indici, partizionamento tabelle audit per retention.
- Entities: Order, Payment, InventoryItem, Shipping, AuditLog, IdempotencyKey.
- Repository pattern con transazioni e locking ottimistico.
- Integrazione Stripe per pagamenti con retry e idempotency.
- Integrazione Logistics REST per spedizioni.
- Notifiche email via SendGrid REST API.
- Consumo messaggi RabbitMQ per eventi ordine asincroni.
- Tutti i dati sensibili criptati a riposo e accessi regolamentati.

## 8. Autenticazione e autorizzazione
- Autenticazione via JWT Bearer Token verificato all’ingresso API.
- Validazione ruoli (es. admin per rimborsi backoffice).
- Nessuna gestione utenti interna, delegata a identity provider esterno.
- Sanitizzazione input per prevenzione injection.
- Controlli role-based per backoffice.
- Gestione sessione stateless via token.

## 9. Gestione errori
- Validazione input: HTTP 400 con errorCode e messaggi dettagliati.
- Autenticazione/Autorizzazione: HTTP 401/403 con motivazione.
- Conflitto idempotenza (replay con payload differente): HTTP 409 Conflict.
- Errori temporanei integrazione: retry automatici con backoff.
- Errori permanenti: aggiornamento stato ordine con failure code (es: PAYMENT_FAILED).
- Dead-letter queue per eventi non processabili.
- Fallimenti audit logging non bloccano utenti ma generano allerta operativa.
- Timeout API con HTTP 503 per servizi non disponibili.

## 10. Strategia di test backend

| Nome test                          | Tipo        | Verifica                                | Input                             | Output atteso                             |
|-----------------------------------|-------------|----------------------------------------|----------------------------------|------------------------------------------|
| CreateOrder_ValidRequest           | Unit        | Validazione payload, persistenza ordine| Payload valido ordine            | 201 Created con orderId e status iniziale|
| CreateOrder_IdempotencyConflict    | Integration | Blocco duplicati con stesso Idempotency-Key | Duplicate request               | 409 Conflict                             |
| GetOrderDetail_ExistingOrder       | Unit        | Recupero dati ordine completo          | orderId esistente                | Dettaglio ordine con stato e pagamenti  |
| PaymentAuthorization_StripeSuccess | Integration | Integrazione Stripe autorizzazione     | Chiamata autorizzazione Stripe  | Stato ordine aggiornato a PAYMENT_AUTHORIZED |
| PaymentCapture_RetryOnTimeout      | Integration | Retry automatico su Stripe capture      | Timeout Stripe durante capture  | Riuscito capture o stato failure gestito|
| ShipmentCreation_LogisticsSuccess  | Integration | Creazione spedizione logistica          | Dati spedizione corretti        | Salvataggio shipmentId e stato SHIPPED  |
| Notification_EmailSendRetry        | Unit        | Retry invio email fallito                | Fallimento invio email          | Log di errore e successivo retry        |
| Refund_Backoffice_AdminRole        | E2E         | Permesso e operazione rimborso           | Operatore admin, request refund | Stato rimborso aggiornato e audit log   |
| EventHandling_Deduplication        | Integration | Deduplicazione eventi RabbitMQ           | Event duplicati in ordine       | Aggiornamento stato ordine coerente     |
| ConcurrencyStock_UpdateConsistency | Integration | Controllo locking ottimistico su stock   | Aggiornamenti concorrenti stock | Nessuna inconsistenza stock              |

## 11. Rischi tecnici
- Ordinamento e deduplicazione eventi RabbitMQ in condizioni di alta concorrenza e rete instabile.
- Gestione concorrenza e locking ottimistico su stock e stato ordine.
- Parametrizzazione retry per evitare spreco risorse o ritardi.
- Compliance GDPR e PCI-DSS da mantenere aggiornata a livello di codice e infrastruttura.
- Timeout e pulizia pagamenti autorizzati non catturati su Stripe.
- Dipendenze da provider esterno per autenticazione backoffice.
- Scalabilità DB e gestione log audit a lungo termine con archiviazione efficiente.
- Rischi di overload e necessità di shard o caching a carichi superiori al progetto.

## 12. Struttura file proposta

```
/src
  /api
    OrderController.java        # Espone REST API ordini e backoffice
    RefundController.java       # Endpoint rimborsi backoffice
  /service
    OrderService.java           # Business logic ordini
    PaymentService.java         # Integrazione Stripe con gestione idempotenza e retry
    LogisticsService.java       # Comunicazione con API logistica e gestione spedizioni
    NotificationService.java    # Invio email e retry tramite SendGrid
    EventProcessor.java         # Consumer RabbitMQ, dedup e ordinamento eventi
    AuditLogService.java        # Persistenza log audit immutabili
  /repository
    OrderRepository.java        # Accesso dati e transazioni ordine
    PaymentRepository.java      # Persistenza pagamenti
    InventoryRepository.java    # Accesso dati stock con locking ottimistico
    ShippingRepository.java     # Persistenza dati spedizioni
    AuditLogRepository.java     # Persistenza log audit
    IdempotencyKeyRepository.java # Gestione token idempotenza
  /model
    Order.java                 # Entity ordine
    Payment.java               # Entity pagamento
    InventoryItem.java         # Entity stock
    Shipping.java              # Entity spedizione
    AuditLog.java              # Log audit
    IdempotencyKey.java        # Token di idempotenza
  /config
    SecurityConfig.java        # Configurazioni JWT, CORS, sanitizzazione input
    RabbitMQConfig.java        # Configurazione consumer eventi
    DatabaseConfig.java        # Pool connessioni e schema PostgreSQL
  /exception
    ApiException.java          # Exception custom per gestione errori API
    RetryException.java        # Exception retry integrato
  /util
    IdempotencyKeyUtil.java    # Utilità per generazione e verifica token idempotenza
    DateTimeUtil.java          # Gestione timestamp ISO8601
```

## 13. Piano di implementazione
1. Setup ambiente sviluppo e repository con struttura base (config, modelli, repository).
2. Implementazione API Ordini: endpoint POST /api/orders con validazione payload e idempotenza.
3. Persistenza ordine con gestione transazioni e locking ottimistico.
4. Integrazione Stripe per autorizzazione e cattura pagamento con retry e logging.
5. Realizzazione modulo gestione inventario e sincronizzazione stock.
6. Implementazione consumer RabbitMQ per eventi ordine asincroni.
7. Integrazione logistica: creazione spedizioni e tracking.
8. Notifiche email via SendGrid e gestione retry.
9. Endpoint backoffice per listing ordini e rimborsi con controllo ruoli.
10. Audit logging per tutte le transizioni di stato e operazioni critiche.
11. Scrittura test unitari, integration ed end-to-end secondo strategia.
12. Effettuare stress test carico per validare performance e scalabilità.
13. Deployment e monitoraggio con log strutturati e allarmi per errori critici.
14. Revisione compliance GDPR e PCI-DSS con audit finale prima rilascio.