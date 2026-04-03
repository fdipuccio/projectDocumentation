MODULE: BACKEND VERSION: 1

## 1. Obiettivo backend
Sviluppare un backend microservizio worker per la gestione ordini e-commerce, con gestione completa del ciclo di vita ordine: creazione, pagamento, stato, spedizione, rimborso e cancellazione. Assicurare scalabilità, resilienza, sicurezza, tracciabilità completa ed integrazione affidabile con servizi esterni quali Stripe (pagamenti), logistica, RabbitMQ (eventi asincroni) e SendGrid (notifiche).

## 2. Assunzioni tecniche
- Architettura microservizio RESTful con JWT Bearer Token per autenticazione e autorizzazione.
- Formato JSON per richieste e risposte API.
- Linguaggio Java con framework Bear per API REST e Data Access.
- PostgreSQL DBMS con transazioni ACID per garantire coerenza su operazioni critiche.
- Uso RabbitMQ per comunicazioni asincrone interne/esterne.
- Integrazione con Stripe per pagamenti (authorization, capture, refund).
- Gestione retry, circuit breaker e idempotenza per resilienza.
- Logging strutturato e audit log immutabile per tracciabilità e compliance GDPR.

## 3. Architettura backend
- Layer API REST esposto in HTTPS con validazione payload e autenticazione token JWT.
- Business logic su moduli separati, implementando state machine per stato ordine.
- Persistenza con repository pattern separato per ordini, pagamenti, stock, audit log.
- Gestione concorrenza con locking DB e transazioni serializzabili.
- Consumer RabbitMQ per eventi asincroni con idempotenza e dead letter queue.
- Integrazione robusta con servizi esterni (Stripe, logistica, SendGrid) con retry e fallback.
- Audit trail dettagliato per ogni transizione di stato ordine.

## 4. Moduli e responsabilità
- API Controller: gestione routing endpoint ordini e backoffice con validazione.
- Order Service: business logic per creazione ordine, gestione pagamento, stato, rimborso.
- Payment Service: integrazione con Stripe per authorization, capture, refund con idempotenza.
- Stock Service: aggiornamento e ripristino stock con transazioni ACID.
- Audit Service: registrazione immutabile transizioni stato.
- RabbitMQ Consumer: elaborazione eventi aggiornamento stato ordine in modo idempotente.
- Notification Service: invio email e push tramite SendGrid e servizi push esterni.
- Security Module: gestione autenticazione e autorizzazione JWT.
- Repository Layer: accesso dati verso PostgreSQL con pattern repository e DAO.

## 5. API principali

### POST /orders 
- Metodo: POST
- Path: /orders
- Request schema:
```json
{
  "customerId": "string",
  "items": [{"productId": "string", "quantity": "integer"}],
  "paymentMethod": {"type": "string", "stripePaymentIntentId": "string"},
  "shippingAddress": {"street": "string", "city": "string", "postalCode": "string", "country": "string"},
  "orderTotal": "number"
}
```
- Response schema:
```json
{
  "orderId": "string",
  "status": "pending_payment",
  "paymentAuthorizationId": "string",
  "createdAt": "ISO-8601 timestamp"
}
```
- Esempio: 
```json
POST /orders
{
  "customerId": "cust123",
  "items": [{"productId": "prod456", "quantity": 2}],
  "paymentMethod": {"type": "stripe_card", "stripePaymentIntentId": "pi_789"},
  "shippingAddress": {"street": "Via Roma 1", "city": "Milano", "postalCode": "20100", "country": "IT"},
  "orderTotal": 100.50
}

Response:
{
  "orderId": "ord123",
  "status": "pending_payment",
  "paymentAuthorizationId": "auth_456",
  "createdAt": "2024-06-01T12:00:00Z"
}
```

### GET /orders/{orderId}
- Metodo: GET
- Path: /orders/{orderId}
- Response schema:
```json
{
  "orderId": "string",
  "customerId": "string",
  "items": [{"productId": "string", "quantity": "integer", "unitPrice": "number"}],
  "orderTotal": "number",
  "status": "string",
  "paymentDetails": {"authorizationId": "string", "captureId": "string", "refundId": "string | null"},
  "shippingAddress": {"street": "string", "city": "string", "postalCode": "string", "country": "string"},
  "auditLog": [{"state": "string", "timestamp": "ISO-8601 timestamp", "changedBy": "string", "details": "string"}],
  "createdAt": "ISO-8601 timestamp",
  "updatedAt": "ISO-8601 timestamp"
}
```
- Esempio risposta:
```json
{
  "orderId": "ord123",
  "customerId": "cust123",
  "items": [{"productId": "prod456", "quantity": 2, "unitPrice": 50.25}],
  "orderTotal": 100.50,
  "status": "paid",
  "paymentDetails": {"authorizationId": "auth_456", "captureId": "cap_789", "refundId": null},
  "shippingAddress": {"street": "Via Roma 1", "city": "Milano", "postalCode": "20100", "country": "IT"},
  "auditLog": [{"state": "paid", "timestamp": "2024-06-01T12:05:00Z", "changedBy": "system", "details": "Payment captured"}],
  "createdAt": "2024-06-01T12:00:00Z",
  "updatedAt": "2024-06-01T12:05:00Z"
}
```

### POST /orders/{orderId}/refund
- Metodo: POST
- Path: /orders/{orderId}/refund
- Request schema:
```json
{
  "reason": "string",
  "refundAmount": "number"
}
```
- Response schema:
```json
{
  "orderId": "string",
  "refundId": "string",
  "status": "refunded",
  "refundedAmount": "number",
  "refundProcessedAt": "ISO-8601 timestamp"
}
```
- Esempio:
```json
POST /orders/ord123/refund
{
  "reason": "Prodotto difettoso",
  "refundAmount": 100.50
}

Response:
{
  "orderId": "ord123",
  "refundId": "refund789",
  "status": "refunded",
  "refundedAmount": 100.50,
  "refundProcessedAt": "2024-06-02T10:00:00Z"
}
```

### GET /orders (Backoffice)
- Metodo: GET
- Path: /orders
- Query Params (optional): status, fromDate, toDate, page, pageSize
- Response schema:
```json
{
  "orders": [
    {
      "orderId": "string",
      "customerId": "string",
      "orderTotal": "number",
      "status": "string",
      "createdAt": "ISO-8601 timestamp",
      "updatedAt": "ISO-8601 timestamp"
    }
  ],
  "pagination": {
    "page": "integer",
    "pageSize": "integer",
    "totalPages": "integer",
    "totalItems": "integer"
  }
}
```

## 6. Business logic
- Validazione input ordini: quantità positive, dati indirizzo completi, prezzo totale corretto, paymentMethod supportato.
- Creazione ordine in stato "pending_payment" e invio autorizzazione pagamento a Stripe.
- Aggiornamento stato ordine in base a esito pagamento (authorized, paid).
- Acquisizione asincrona del capture pagamento e scalatura stock in transazione ACID.
- Gestione rimborsi manuali con chiamata a Stripe refund e ripristino stock.
- Gestione eventi asincroni da RabbitMQ per aggiornamenti stato ordine con idempotenza.
- Audit log di ogni transizione di stato ordine per compliance e troubleshooting.
- Notifiche email/push ad ogni cambio stato tramite servizi esterni.

## 7. Persistenza e integrazioni
- PostgreSQL con schema per ordini, pagamenti, stock, audit log.
- Transazioni ACID per garantire coerenza in operazioni di pagamento e stock.
- Integrazione Stripe per authorization, capture, refund con idempotenza e retry.
- RabbitMQ consumer per eventi ordine con gestione idempotente e dead-letter queue.
- API REST HTTPS per integrazione logistica e servizi notifiche (SendGrid).
- Backup e retention audit log conformi GDPR.

## 8. Autenticazione e autorizzazione
- JWT Bearer token obbligatorio con ruoli e permessi.
- Ruolo "operator" per API backoffice.
- Token validato su ogni chiamata API; errori 401 Unauthorized o 403 Forbidden.
- Comunicazioni via HTTPS.
- Nessuna gestione autenticazione cliente finale (delegata a sistemi esterni).

## 9. Gestione errori
- Errori validazione input: HTTP 400 con dettagli.
- Errori autenticazione autorizzazione: 401, 403.
- Errori interni: 500 con messaggi generici.
- Errori integrazione esterni (es. Stripe, Logistica, SendGrid) gestiti con retry, circuit breaker e rollback transazioni.
- Idempotenza per evitare duplicati.
- Logging dettagliato e audit errori critici.
- Response error JSON: 
```json
{
  "errorCode": "string",
  "message": "string",
  "details": "optional string or object"
}
```

## 10. Strategia di test backend
- Test unitari Service Layer: verifica validazione dati, logica di business, transizioni stato ordine.
- Test integration su Repository: verifica transazioni DB, locking, corretto salvataggio dati ordini, pagamenti, audit.
- Test endpoint API (integration/end-to-end):
  1. Creazione ordine valido: verifica response, stato ordine, chiamata Stripe.
  2. Creazione ordine invalid input: riceve 400 con errori.
  3. Rimborso manuale: verifica esito refund e aggiornamenti DB.
  4. Consumer RabbitMQ: verifica elaborazione evento, idempotenza.
  5. Autenticazione API: verifica accesso con token valido e rifiuto senza token.
- Test resilienza: simulazione errori Stripe e reti con retry.
- Copertura test > 85%, con pipeline CI/CD per esecuzione automatica.

## 11. Rischi tecnici
- Race condition su aggiornamenti stock ad alta concorrenza, mitigata da transazioni serializzabili.
- Dipendenza da servizi esterni con possibili downtime, mitigati da circuit breaker e retry.
- Locking e rallentamenti DB su picchi elevati.
- Gestione idempotenza critica per evitare incoerenze pagamenti e eventi.
- Compliance GDPR su retention audit log.
- Scadenze token JWT necessitano gestione refresh.
- Complessità workflow ordini con numerosi stati e transizioni, richiede test approfonditi.
- Esclusione ordini parziali semplifica MVP ma limita funzionalità iniziali.

## 12. Struttura file proposta
```
src/
 ├─ controller/
 │    ├─ OrderController.java         # API REST endpoints gestione ordini e backoffice
 │    └─ RefundController.java        # Endpoint rimborso manuale
 ├─ service/
 │    ├─ OrderService.java            # Logica business gestione ordini e stato
 │    ├─ PaymentService.java          # Integrazione e gestione Stripe
 │    ├─ StockService.java            # Gestione stock prodotti con transazioni
 │    ├─ AuditService.java            # Registrazione log audit immutabili
 │    ├─ NotificationService.java     # Integrazione notifiche email/push
 │    └─ RabbitMQConsumer.java        # Consumatore eventi RabbitMQ aggiornamenti
 ├─ repository/
 │    ├─ OrderRepository.java
 │    ├─ PaymentRepository.java
 │    ├─ StockRepository.java
 │    └─ AuditLogRepository.java
 ├─ security/
 │    └─ JwtAuthFilter.java           # Middleware autenticazione JWT
 ├─ config/
 │    └─ ApplicationConfig.java       # Configurazioni generali sistema e DB
 ├─ model/
 │    ├─ Order.java                   # Modelli dominio ordine
 │    ├─ Payment.java                 # Modelli dominio pagamento
 │    ├─ StockItem.java               # Modelli dominio stock
 │    ├─ AuditLogEntry.java           # Modelli dominio audit log
 │    └─ DTOs.java                   # Classi DTO request/response API
 └─ util/
      └─ IdempotencyUtil.java         # Utility supporto idempotenza operazioni
```

## 13. Piano di implementazione
1. Setup ambiente sviluppo, DB PostgreSQL con schema base.
2. Implementazione layer repository e modelli dominio.
3. Sviluppo servizi core: OrderService, PaymentService, StockService.
4. Implementazione API REST base per creazione ordine (POST /orders) e recupero (GET /orders/{orderId}).
5. Integrazione Stripe per authorization, capture e refund, con test.
6. Sviluppo consumer RabbitMQ per gestione eventi asincroni stato ordine.
7. Implementazione backoffice API GET /orders e POST /orders/{orderId}/refund.
8. Implementazione logging strutturato e audit log.
9. Sviluppo NotificationService per invio email e push.
10. Implementazione sicurezza JWT con middleware.
11. Scrittura test unitari, integration e e2e.
12. Deployment ambiente test e collaudo con simulazioni carico e fault injection.
13. Migliorie e ottimizzazioni performance, gestione errori e scalabilità.
14. Rilascio produzione con monitoraggio continuo e alert.

---

*Documento generato conforme specifiche funzionali e linee guida architetturali, pronto per validazione QA e avvio implementazione.*