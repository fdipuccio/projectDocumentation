MODULE: BACKEND VERSION: 1

## 1. Obiettivo backend
Il backend è un microservizio worker per la gestione ordini di un sistema e-commerce. L’obiettivo è gestire l’intero ciclo di vita degli ordini, dall’acquisizione tramite API alla gestione asincrona degli stati e pagamenti, con integrazione affidabile con servizi esterni quali Stripe per i pagamenti, un servizio di logistica esterno e RabbitMQ per eventi asincroni. Il sistema garantisce transazioni ACID, idempotenza e conformità audit tramite un log immutabile.

## 2. Assunzioni tecniche
- Il microservizio si basa su architettura a microservizi e pattern event-driven.
- Comunicazione tramite API REST JSON su HTTPS con autenticazione JWT.
- Persistenza dati su PostgreSQL con transazioni ACID.
- Uso di message broker RabbitMQ per eventi asincroni.
- Gestione integrata di pagamenti con Stripe, notifiche con SendGrid o servizi push.
- Middleware di sicurezza per validazione token e autorizzazioni basate su ruoli.
- Garantita idempotenza in operazioni critiche: pagamenti, eventi, notifiche.
- Il sistema deve supportare un carico di circa 500 ordini/minuto con latenza sotto 200ms (95° percentile).

## 3. Architettura backend
Il backend è organizzato in moduli layered:
- API REST Controller: espone endpoint per gestione ordini lato cliente e backoffice operatori.
- Order Service: orchestrazione flussi di stato ordini con state machine, gestisce pagamenti, stock, notifiche e audit log.
- Integration Layer: moduli dedicati per Stripe, logistica, RabbitMQ e servizi notifiche.
- Persistence Layer: database PostgreSQL con repository ORM gestisce entità ordine, pagamento, stock, audit log, spedizioni.
- Security Middleware: autenticazione JWT e autorizzazioni ruoli.
- Messaging Consumer: riceve eventi RabbitMQ per aggiornamenti asincroni degli ordini.
- Notification Service: invio email/push con gestione opt-out.

## 4. Moduli e responsabilità
- **API REST Controller:** gestione routing, validazione input/output JSON, autorizzazione.
- **Order Service:** logica di business ordine, transizioni stati, gestione pagamenti.
- **Repositories:** interfacce per accesso transazionale dati su Ordini, Pagamenti, Stock, AuditLog e Spedizioni.
- **Integration Clients:** adapter per Stripe (authorization, capture, refund), Logistica REST, RabbitMQ consumer/producer, SendGrid/Push notifications.
- **Security Middleware:** token JWT parse e verifica permessi.
- **Event Processor:** elaborazione eventi RabbitMQ con idempotenza.
- **Notification Service:** orchestrazione invio notifiche e logging.

## 5. API principali

### 1. POST /api/orders
- Metodo HTTP: POST
- Path: /api/orders
- Request schema:
```json
{
  "customer_id": "uuid",
  "items": [
    {
      "product_id": "uuid",
      "quantity": "integer > 0"
    }
  ],
  "shipping_address": {
    "street": "string",
    "city": "string",
    "postcode": "string",
    "country": "string"
  },
  "payment_method": {
    "type": "card",
    "card_token": "string"
  }
}
```
- Response schema (201 Created):
```json
{
  "order_id": "uuid",
  "status": "Created",
  "created_at": "timestamp"
}
```
- Esempio concreto:
```json
{
  "order_id": "123e4567-e89b-12d3-a456-426614174000",
  "status": "Created",
  "created_at": "2024-06-01T12:00:00Z"
}
```

### 2. GET /api/orders/{order_id}
- Metodo HTTP: GET
- Path: /api/orders/{order_id}
- Response: dettagli ordine con stato, items, pagamenti; 404 se non trovato; autorizzazione necessaria.

### 3. POST /api/orders/{order_id}/capture
- Metodo HTTP: POST
- Path: /api/orders/{order_id}/capture
- Request Body: vuoto
- Response schema (200 OK):
```json
{
  "order_id": "uuid",
  "status": "PaymentCaptured",
  "captured_at": "timestamp"
}
```

### 4. POST /api/orders/{order_id}/cancel
- Metodo HTTP: POST
- Path: /api/orders/{order_id}/cancel
- Request Body: vuoto
- Response: conferma annullamento e rilascio stock.

### 5. GET /api/backoffice/orders
- Metodo HTTP: GET
- Path: /api/backoffice/orders
- Query params: filtri su stato, data, cliente
- Response: lista paginata ordini
- Autenticazione JWT con ruolo operatore obbligatoria

### 6. GET /api/backoffice/orders/{order_id}
- Metodo HTTP: GET
- Path: /api/backoffice/orders/{order_id}
- Response: dettaglio completo ordine con pagamenti e audit log

### 7. POST /api/backoffice/orders/{order_id}/refund
- Metodo HTTP: POST
- Path: /api/backoffice/orders/{order_id}/refund
- Request schema:
```json
{
  "reason": "string"
}
```
- Response schema (200 OK):
```json
{
  "order_id": "uuid",
  "status": "Refunded",
  "refunded_at": "timestamp"
}
```

## 6. Business logic
- Creazione ordine: validazione input, salvataggio transazionale ordine e items, chiamata Stripe per authorization pagamento con token idempotenza, aggiornamento stato ordine, gestione retry e notifiche.
- Captura pagamento: eseguito solo se stato ordine “Authorized”, chiamata capture Stripe, aggiornamento stock, transazione e audit.
- Aggiornamento stato tramite eventi RabbitMQ: idempotenza verifica event_id, aggiornamento transazionale stato ordine, audit log, chiamate a logistica e notifiche.
- Annullamento ordine: transazione di rilascio stock, aggiornamento stato ordine.
- Rimborso manuale da backoffice: verifica permessi, chiamata refund Stripe con idempotenza, update stato e stock, audit log e notifiche.

## 7. Persistenza e integrazioni
- Persistenza con PostgreSQL, schema normalizzato con tabelle per Order, OrderItem, Payment, Stock, AuditLog e ShippingInfo.
- Transazioni ACID per operazioni critiche.
- Repository pattern e ORM Java per astrazione DB.
- Integrazione Stripe per pagamenti con retry e idempotenza.
- RabbitMQ consumer per gestione eventi asincroni, con deduplicazione eventi.
- Integrazione REST con servizio logistico esterno, retry e circuit breaker.
- Servizio notifiche email/push tramite SendGrid e wrapper push con supporto opt-out.

## 8. Autenticazione e autorizzazione
- Tutte le API protette richiedono JWT Bearer token nell’header Authorization.
- Token JWT contengono ruoli e permessi per abilitare accesso specifico (es. backoffice).
- Middleware di sicurezza verifica validità token, ruoli e scope.
- Accessi non autorizzati rispondono 401/403.
- Comunicazione sempre su HTTPS.

## 9. Gestione errori
- Validazioni input rispondono 400 Bad Request con dettagli JSON.
- Errori di autorizzazione 401 Unauthorized o 403 Forbidden.
- Errori di conflitti di stato 409 Conflict.
- Errori server e eccezioni loggate e risposte 500 Internal Server Error.
- Gestione errori esterni con ri-try e backoff esponenziale, fallback e dead-letter queue per messaging.
- Idempotenza assicura coerenza dati in caso di retry.
- Logging strutturato JSON con stack trace e contesto per troubleshooting.

## 10. Strategia di test backend

| Nome Test                          | Tipo        | Cosa verifica                                     | Input                             | Output Atteso                         |
|-----------------------------------|-------------|--------------------------------------------------|----------------------------------|-------------------------------------|
| CreateOrder_ValidInput             | Unit        | Validazione creazione ordine corretto            | Request POST /api/orders valido   | 201 Created con order_id e stato    |
| CreateOrder_InvalidInput           | Unit        | Gestione errori validazione                       | Request POST /api/orders errato   | 400 Bad Request con dettagli errori|
| CapturePayment_Successful          | Integration | Transizione stato ordine Authorized->Captured     | POST /api/orders/{id}/capture    | 200 OK con status PaymentCaptured   |
| CapturePayment_InvalidState        | Integration | Errore se stato ordine non Authorized             | Stato ordine diverso              | 409 Conflict                        |
| RefundOrder_Backoffice_Success     | Integration | Rimborso manuale ordine con rollback stock       | POST /api/backoffice/orders/{id}/refund con reason | 200 OK stato Refunded               |
| RabbitMQEventProcessing_Idempotency| Integration | Gestisce duplicati eventi RabbitMQ correttamente | Evento duplicato con event_id     | Aggiornamento ordine idempotente    |
| AuthenticationMiddleware_InvalidToken | Unit    | Blocca richieste senza token o token invalidi     | Richiesta senza token             | 401 Unauthorized                    |
| AuthorizationMiddleware_MissingRole| Unit       | Blocca accesso utenti senza permessi corretti     | Token senza ruolo backoffice      | 403 Forbidden                      |

## 11. Rischi tecnici
- Race conditions sulla gestione stock in concorrenza elevata.
- Downtime o latenza elevata servizi esterni (Stripe, logistici, notifiche).
- Gestione scadenza token JWT e refresh.
- Errori nell’idempotenza possono causare duplicazioni o stati inconsistente.
- Crescita volume dati audit log e gestione retention.
- Scalabilità e tuning delle risorse DB e consumer eventi.
- Compliance GDPR per dati sensibili di pagamento.
- Gestione ruoli backoffice per evitare violazioni di sicurezza.

## 12. Struttura file proposta
```
backend/
│
├── api/
│   ├── OrderController.java      # Espone API REST ordini (cliente)
│   ├── BackofficeController.java # Espone API REST backoffice
│   └── SecurityMiddleware.java   # Middleware JWT e autorizzazioni
│
├── service/
│   ├── OrderService.java         # Logica business ordine e workflow
│   ├── PaymentService.java       # Gestione pagamento Stripe
│   ├── StockService.java         # Gestione stock e locking concorrenza
│   ├── NotificationService.java  # Invio email/push e gestione opt-out
│   └── EventProcessor.java       # Consumer RabbitMQ eventi
│
├── repository/
│   ├── OrdersRepository.java
│   ├── PaymentRepository.java
│   ├── StockRepository.java
│   ├── AuditLogRepository.java
│   └── ShippingRepository.java
│
├── integration/
│   ├── StripeClient.java         # Wrapper chiamate Stripe (auth, capture, refund)
│   ├── LogisticsClient.java      # Wrapper chiamate REST logistica
│   ├── RabbitMQConsumer.java     # Consumer RabbitMQ eventi
│   └── NotificationClient.java   # Wrapper invio SendGrid e push
│
├── model/
│   ├── Order.java                # Entity ordine
│   ├── OrderItem.java
│   ├── Payment.java
│   ├── Stock.java
│   ├── AuditLog.java
│   └── ShippingInfo.java
│
├── exceptions/
│   ├── ValidationException.java
│   ├── AuthorizationException.java
│   └── IntegrationException.java
│
└── utils/
    ├── IdempotencyHelper.java   # Gestione token idempotenza
    └── JsonLogger.java          # Logger strutturato JSON
```

## 13. Piano di implementazione
1. Setup progetto con struttura base, dipendenze e configurazione database.
2. Implementazione entità e repository (Order, Payment, Stock, AuditLog).
3. Sviluppo API REST Controller per ordini e backoffice con middleware sicurezza.
4. Implementazione logica Ordine (state machine workflow) e servizi di pagamento Stripe.
5. Integrazione layer con Stripe, logistica, RabbitMQ, SendGrid.
6. Implementazione consumer eventi RabbitMQ con idempotenza.
7. Gestione notifiche e opt-out integration.
8. Test unitari e di integrazione su singoli moduli.
9. Test end-to-end su flussi business principali (creazione, pagamento, refund).
10. Deployment ambiente test e staging.
11. Monitoraggio, tuning performance e gestione escalation incidenti.
12. Documentazione finale e training team supporto.