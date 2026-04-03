MODULE: BACKEND VERSION: 1

## 1. Obiettivo backend
Implementare un microservizio backend worker per la gestione ordini e-commerce che garantisca la creazione, gestione, aggiornamento, pagamento e rimborso degli ordini con integrazione a servizi esterni (Stripe per pagamenti, RabbitMQ per eventi, integrazione logistica e notifiche SendGrid), assicurando sicurezza, transazioni ACID, idempotenza, resilienza e compliance GDPR.

## 2. Assunzioni tecniche
- Stack: Java con Bear framework.
- Database: PostgreSQL per persistenza relazionale ACID.
- Messaggistica: RabbitMQ per eventi asincroni.
- Integrazioni esterne: Stripe per pagamenti, servizio logistico esterno REST/async, SendGrid per email e push.
- Autenticazione e autorizzazione JWT Bearer Token.
- Comunicazioni su HTTPS.
- Pattern event-driven con gestione retry e circuit breaker.
- Idempotenza estesa per operazioni critiche (ordine, pagamento, rimborso, eventi).
- Logging strutturato per audit e monitoraggio.
- Compliance normativa (es. GDPR, retention audit log).

## 3. Architettura backend
Architettura microservizio a moduli layered con componenti:
- API REST per ricezione e consultazione ordini.
- Modulo dominio con business logic (ordine, pagamento, rimborso, stock).
- Layer persistenza con repository JPA/PostgreSQL.
- Modulo integrazione per Stripe, RabbitMQ, logistica, SendGrid (SDK e REST).
- Bus eventi asincrono per aggiornamento stato ordine.
- Middleware autenticazione JWT e autorizzazione ruoli.
- Gestione errori centralizzata con mapping su codici HTTP.
- Worker asincrono per elaborazioni background e consumi eventi.
- Sistema audit log append-only.

## 4. Moduli e responsabilità
- API Controller: espone endpoint REST sicuri.
- Order Service: gestione creazione ordine, stato pagamento, rimborso, transizioni stato.
- Payment Service: orchestrazione chiamate Stripe con idempotenza e retry.
- Stock Service: gestione decremento/incremento stock con locking.
- Audit Log Service: scrittura immutabile delle transizioni e eventi.
- RabbitMQ Consumer/Producer: eventi ordine e notifiche.
- Notification Service: invio email/push via SendGrid con retries.
- Auth Middleware: validazione JWT e controllo ruoli.
- Persistence Layer: repository per dominio dati e gestione transazioni.

## 5. API principali

### 5.1 Creazione Ordine
- Metodo HTTP: POST
- Path: /api/orders
- Request body:
```json
{
  "customerId": "string",
  "items": [
    {
      "productId": "string",
      "quantity": integer
    }
  ],
  "paymentMethod": "stripe",
  "shippingAddress": {
    "street": "string",
    "city": "string",
    "postalCode": "string",
    "country": "string"
  }
}
```
- Response 201 Created:
```json
{
  "orderId": "string",
  "status": "PENDING_PAYMENT",
  "createdAt": "ISO8601 datetime"
}
```
- Errori: 400, 401, 500

### 5.2 Recupero Dettaglio Ordine
- Metodo HTTP: GET
- Path: /api/orders/{orderId}
- Response 200 OK:
```json
{
  "orderId": "string",
  "customerId": "string",
  "items": [
    {
      "productId": "string",
      "quantity": integer,
      "price": "decimal"
    }
  ],
  "paymentStatus": "AUTHORIZED | CAPTURED | REFUNDED",
  "orderStatus": "NEW | PENDING_PAYMENT | PAID | SHIPPED | COMPLETED | CANCELED",
  "shippingAddress": {...},
  "createdAt": "ISO8601 datetime",
  "updatedAt": "ISO8601 datetime",
  "auditLog": [
    {
      "eventId": "string",
      "previousStatus": "string",
      "newStatus": "string",
      "timestamp": "ISO8601 datetime",
      "operatorId": "string | null"
    }
  ]
}
```
- Errori: 404, 401

### 5.3 Lista Ordini con Filtri
- Metodo HTTP: GET
- Path: /api/orders
- Query Params: status (opzionale), page, pageSize
- Response 200 OK:
```json
{
  "orders": [
    {
      "orderId": "string",
      "customerId": "string",
      "orderStatus": "string",
      "paymentStatus": "string",
      "createdAt": "ISO8601 datetime"
    }
  ],
  "pagination": {
    "currentPage": integer,
    "totalPages": integer,
    "totalItems": integer
  }
}
```
- Accesso: backoffice operatori
- Errori: 401, 403

### 5.4 Rimborso Manuale
- Metodo HTTP: POST
- Path: /api/orders/{orderId}/refund
- Request body: {}
- Response 200 OK:
```json
{
  "orderId": "string",
  "refundStatus": "COMPLETED",
  "refundProcessedAt": "ISO8601 datetime"
}
```
- Errori: 400 (non rimborso), 404, 401, 403

### 5.5 Recupero Stato Ordine
- Metodo HTTP: GET
- Path: /api/orders/{orderId}/status
- Response 200 OK:
```json
{
  "orderId": "string",
  "currentStatus": "string",
  "paymentStatus": "string",
  "lastUpdated": "ISO8601 datetime"
}
```

## 6. Business logic
- Creazione ordine con stato iniziale PENDING_PAYMENT.
- Chiamata a Stripe per autorizzazione fondi (idempotent key).
- Transizione stato ordine secondo flusso pagamento (AUTHORIZED, PAID).
- Gestione decremento stock al pagamento confermato, rollback su errori.
- Pubblicazione eventi stato ordine su RabbitMQ per integrazione logistica e notifiche.
- Gestione rimborso manuale da backoffice: verifica stato, chiamata Stripe refund, rollback stock, audit log update.
- Consumatori eventi idempotenti per aggiornamento stato ordine e notifica utenti.
- Gestione audit log immutabile per ogni transizione di stato e evento rilevante.
- Retry automatico per chiamate esterne (Stripe, logistica, SendGrid) con circuit breaker.

## 7. Persistenza e integrazioni
- PostgreSQL con transazioni ACID per operazioni critiche.
- Repository per aggregati: Order, Payment, Stock, AuditLog, Logistics.
- Uso di locking pessimista per decremento stock (SELECT FOR UPDATE).
- Append-only table per audit log.
- Stripe SDK per pagamenti con idempotenza e retry.
- RabbitMQ per eventi asincroni, consumer idempotenti, dead letter queue.
- Integrazione con servizio logistica via REST/async con retry e circuit breaker.
- SendGrid SDK per notifiche email/push, retry su errori transitori.

## 8. Autenticazione e autorizzazione
- JWT Bearer Token obbligatorio su tutte le API.
- Middleware Bear framework valida token JWT prima esecuzione business logic.
- Ruoli: "operator" per accesso backoffice e operazioni sensibili.
- Errori: 401 token mancante/invalido, 403 permessi insufficienti.
- Comunicazione solo su HTTPS.

## 9. Gestione errori
- 400 Bad Request per validazione input con messaggi chiari.
- 401/403 per autenticazione/autorizzazione.
- 404 per risorse assenti.
- 409 o 503 per errori payment con possibilità di retry.
- 500 Internal Server Error per errori server e integrazioni esterne.
- Logging dettagliato e audit di errori critici.
- Operazioni idempotenti per evitare duplicazioni stati incoerenti.
- Retry automatico con backoff esponenziale su integrazioni esterne.
- Circuit breaker per prevenire interruzioni a cascata.

## 10. Strategia di test backend

### 10.1 Test di creazione ordine valido
- Tipo: Unit
- Verifica validazione input e creazione ordine con stato iniziale.
- Input: richiesta POST /api/orders con payload corretto.
- Output: ordine creato, risposta 201, stato PENDING_PAYMENT.

### 10.2 Test autorizzazione Stripe
- Tipo: Integration
- Verifica chiamata a Stripe con idempotency key e gestione risposta.
- Input: dati pagamento associati ordine.
- Output: stato ordine aggiornato AUTHORIZED o gestione errori.

### 10.3 Test decremento stock concorrente
- Tipo: Integration
- Verifica locking e decremento stock con richieste parallele.
- Input: diverse richieste di pagamento simultanee.
- Output: stock coerente senza sovrascatti.

### 10.4 Test rimborso manuale
- Tipo: E2E
- Verifica processo rimborso ordine da backoffice, rollback stock, aggiornamento stato.
- Input: chiamata POST /api/orders/{orderId}/refund su ordine rimborsabile.
- Output: risposta 200 con stato REFUNDED e variazioni corrette.

### 10.5 Test autenticazione e autorizzazione API
- Tipo: Unit/Integration
- Verifica middleware JWT e controllo ruoli.
- Input: richieste con token valido, scaduto, mancante, ruolo insufficiente.
- Output: risposte 200, 401, 403 appropriate.

### 10.6 Test gestione errori e retry
- Tipo: Integration
- Simula errori temporanei in Stripe/logistica/SendGrid.
- Verifica retry automatico e circuit breaker.
- Output: corretta gestione errori e logging.

## 11. Rischi tecnici
- Race condition sul decremento stock ad alta concorrenza.
- Downtime o errori servizi esterni (Stripe, logistica, SendGrid).
- Gestione corretta e tempestiva dei token JWT.
- Complessità idempotenza in flussi asincroni e retry.
- Compliance audit log e retention dati.
- Performance sotto carico con locking DB e backlog messaggi.
- Ambiguità requisiti notifiche e logistica.
- Sicurezza app ed esposizione dati sensibili.

## 12. Struttura file proposta

```
/src
 |-- /api
 |     |-- OrderController.java          # Espone API REST ordini
 |     |-- AuthMiddleware.java           # Middleware JWT e autorizzazioni
 |
 |-- /service
 |     |-- OrderService.java             # Logica business ordine, transizioni
 |     |-- PaymentService.java           # Gestione pagamento Stripe
 |     |-- StockService.java             # Gestione stock con locking
 |     |-- RefundService.java            # Gestione rimborso manuale
 |     |-- NotificationService.java     # Invio email/push SendGrid
 |     |-- AuditLogService.java          # Scrittura append-only log eventi
 |     |-- EventConsumer.java            # Consumatore RabbitMQ eventi ordine
 |
 |-- /repository
 |     |-- OrderRepository.java          # CRUD e query ordine, transazioni
 |     |-- PaymentRepository.java        # Persistenza pagamenti e idempotency
 |     |-- StockRepository.java          # Persistenza stock con locking
 |     |-- AuditLogRepository.java       # Persistenza eventi immutabili
 |     |-- LogisticsRepository.java     # Persistenza tracking logistico
 |
 |-- /integration
 |     |-- StripeClient.java             # Wrapper SDK Stripe con retry, idempotency
 |     |-- RabbitMQClient.java           # Producer/consumer RabbitMQ con retry
 |     |-- LogisticsClient.java          # Integrazione servizio logistico
 |     |-- SendGridClient.java           # Client invio notifiche
 |
 |-- /model
 |     |-- Order.java                    # Entità Order dominio
 |     |-- OrderItem.java                # Items ordine
 |     |-- Payment.java                  # Pagamento e stato
 |     |-- Stock.java                    # Stock SKU
 |     |-- AuditLogEntry.java            # Evento audit log
 |     |-- ShipmentTracking.java         # Informazioni logistica
 |
 |-- application.properties             # Configurazioni database, integrazioni, sicurezza
 |-- MainApplication.java               # Avvio applicazione Bear framework
```

## 13. Piano di implementazione

1. Setup ambiente sviluppo, repository codice, pipeline CI/CD.
2. Implementare modello dominio e mapping JPA.
3. Realizzare repository e transazioni PostgreSQL con locking stock.
4. Sviluppo API REST (OrderController) e middleware JWT.
5. Implementazione logica creazione ordine, validazione input, stato iniziale.
6. Integrazione Stripe per autorizzazione e pagamento con idempotency.
7. Gestione RabbitMQ per pubblicazione e consumo eventi aggiornamento stato.
8. Implementazione audit log append-only.
9. Sviluppo flusso rimborso manuale e rollback stock.
10. Integrazione servizi logistici e notifiche SendGrid.
11. Testing unitari, integration e E2E per copertura completa.
12. Messa in produzione fase MVP con monitoraggio e alert.
13. Refactoring e ottimizzazioni performance in base a feedback.

---

**Proposta backend microservizio worker e-commerce gestione ordini completa, coerente con requisiti e stack indicati, pronta per fase di QA.**