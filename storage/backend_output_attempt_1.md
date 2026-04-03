```markdown
MODULE: BACKEND 
VERSION: 1

## 1. Obiettivo backend ##
Il backend è un microservizio di tipo worker dedicato alla gestione ordini di un e-commerce. Gestisce la creazione, il monitoraggio e l’aggiornamento dello stato degli ordini, integrandosi con servizi esterni di pagamento (Stripe), logistica, notifiche (SendGrid, Push) e un sistema di messaggistica asincrona (RabbitMQ). Il sistema garantisce elevata affidabilità, scalabilità, sicurezza, idempotenza e tracciabilità completa delle operazioni.

## 2. Assunzioni tecniche ##
- Linguaggio Java con framework Bear per la gestione API, transazioni e middleware.
- Database relazionale PostgreSQL per persistenza dati con supporto ACID e locking ottimizzato.
- Comunicazione RESTful API con JSON.
- Autenticazione JWT con ruoli differenziati (USER, BACKOFFICE_OPERATOR).
- HTTPS obbligatorio per tutte le chiamate.
- RabbitMQ usato per eventi asincroni e aggiornamenti di stato.
- Integrazione resiliente con Stripe, logistica, SendGrid con retry e circuit breaker.
- Logging strutturato e audit log con correlazione delle richieste per traceability.
- Compliance GDPR per la gestione dati sensibili.
- Idempotenza su tutte le operazioni critiche (es. creazione ordine, refund).

## 3. Architettura backend ##
Architettura multilayer:
- Controller: gestione API REST, validazione input, autorizzazioni.
- Service: logica di business, orchestrazione workflow, rollback.
- Repository: accesso a dati e persistence, gestione transazioni.
- Moduli esterni wrapper: Stripe, Logistica, Notifiche, RabbitMQ.
- Middleware sicurezza (JWT, HTTPS enforcement).
- Event consumer RabbitMQ per aggiornamenti asincroni con gestione idempotenza.

## 4. Moduli e responsabilità ##
- Order Module: gestione ordini, flussi di stato, idempotenza.
- Payment Module: interazione Stripe, autorizzazione, rimborso.
- Shipment Module: integrazione con servizio logistico.
- Stock Module: gestione quantitativi magazzino e sincronizzazione.
- Audit Module: logging dettagliato delle transizioni e operazioni.
- Notification Module: gestione email (SendGrid) e push asincroni.
- Authentication & Authorization Middleware: JWT validation e controllo ruoli.
- RabbitMQ Consumer: ricezione e processing eventi asincroni con retry.
- Error Handling Module: gestione retry, rollback, circuit breaker.

## 5. API principali ##

### POST /api/orders
- Metodo: POST
- Path: /api/orders
- Autenticazione: opzionale (valida dati, idempotencyKey obbligatoria)
- Request Schema:
```json
{
  "customerId": "string",
  "items": [
    {
      "productId": "string",
      "quantity": 1
    }
  ],
  "paymentMethod": "string",
  "shippingAddress": {
    "street": "string",
    "city": "string",
    "postalCode": "string",
    "country": "string"
  },
  "idempotencyKey": "string"
}
```
- Response Schema 201 Created:
```json
{
  "orderId": "string",
  "status": "PENDING_PAYMENT",
  "createdAt": "ISO8601 timestamp"
}
```
- Esempio concreto request:
```json
{
  "customerId": "cust123",
  "items": [
    {"productId": "prod456", "quantity": 2}
  ],
  "paymentMethod": "stripe_card",
  "shippingAddress": {
    "street": "Via Roma 1",
    "city": "Milano",
    "postalCode": "20100",
    "country": "IT"
  },
  "idempotencyKey": "abc-123-def-456"
}
```

### GET /api/orders/{orderId}
- Metodo: GET
- Path: /api/orders/{orderId}
- Autenticazione: JWT (USER)
- Response Schema 200 OK:
```json
{
  "orderId": "string",
  "customerId": "string",
  "items": [
    {
      "productId": "string",
      "quantity": 1,
      "price": 9.99
    }
  ],
  "totalAmount": 29.97,
  "status": "string",
  "paymentStatus": "AUTHORIZED|CAPTURED|REFUNDED|FAILED",
  "shipping": {
    "trackingNumber": "string",
    "carrier": "string",
    "status": "string"
  },
  "createdAt": "ISO8601 timestamp",
  "updatedAt": "ISO8601 timestamp"
}
```
- Esempio concreto response:
```json
{
  "orderId": "order789",
  "customerId": "cust123",
  "items": [
    {"productId": "prod456", "quantity": 2, "price": 9.99}
  ],
  "totalAmount": 19.98,
  "status": "PAYMENT_CONFIRMED",
  "paymentStatus": "CAPTURED",
  "shipping": {
    "trackingNumber": "TRACK123",
    "carrier": "DHL",
    "status": "IN_TRANSIT"
  },
  "createdAt": "2024-06-01T10:00:00Z",
  "updatedAt": "2024-06-01T12:00:00Z"
}
```

### GET /api/orders
- Metodo: GET
- Path: /api/orders
- Autenticazione: JWT (USER)
- Funzionalità: ritorna lista ordini cliente filtrabile/paginabile
- Request parametri tipici: page, size, status

### GET /api/backoffice/orders
- Metodo: GET
- Path: /api/backoffice/orders
- Autenticazione: JWT ruolo BACKOFFICE_OPERATOR
- Funzionalità: lista ordini completa/backoffice con filtri e paginazione

### GET /api/backoffice/orders/{id}
- Metodo: GET
- Path: /api/backoffice/orders/{id}
- Autenticazione: JWT ruolo BACKOFFICE_OPERATOR
- Funzionalità: dettaglio ordine per backoffice

### POST /api/backoffice/orders/{id}/refund
- Metodo: POST
- Path: /api/backoffice/orders/{id}/refund
- Autenticazione: JWT ruolo BACKOFFICE_OPERATOR
- Request Schema:
```json
{
  "reason": "string",
  "confirmedBy": "operatorId",
  "idempotencyKey": "string"
}
```
- Response Schema 200 OK:
```json
{
  "orderId": "string",
  "refundStatus": "REQUESTED|COMPLETED|FAILED",
  "refundAmount": 29.97,
  "createdAt": "ISO8601 timestamp"
}
```

### POST /api/internal/rabbitmq/callback
- Metodo: POST
- Path: /api/internal/rabbitmq/callback
- Autenticazione: interna/autenticata (IP whitelist/JWT custom)
- Funzione: consumatore eventi aggiornamento stato ordine da RabbitMQ, idempotente.

## 6. Business logic ##
- Validazione schema e dati in input e idempotenceKey.
- Verifica stock e disponibilità in modo atomico.
- Autorizzazione pagamento Stripe, con rollback in caso di errore.
- Aggiornamento stato ordine tramite stato macchina e notifiche.
- Creazione spedizione integrata lato logistico post pagamento.
- Gestione asincrona tramite messaggistica RabbitMQ per aggiornamenti e retry automatici.
- Gestione rimborso manuale da backoffice con verifica idempotenza.
- Audit dettagliato di tutte le modifiche di stato.
- Gestione retry, circuit breaker e rollback transazionali per garantire consistenza.
- Compliance GDPR e masking dati sensibili.

## 7. Persistenza e integrazioni ##
- PostgreSQL per dati core: tabelle orders, order_states, payments, shipments, stock_items, audit_logs.
- Transazioni ACID per garantire consistenza.
- Unicità su idempotencyKey per evitare doppie operazioni.
- Locking ottimizzato per gestione concorrente stock e stati ordine.
- Wrapper dedicati per Stripe (pagamenti), servizio logistico (spedizioni), SendGrid e sistema push (notifiche).
- RabbitMQ configurato per eventi asincroni con gestione dead-letter e retry.
- Indici e partizionamenti configurati per performance e scalabilità.

## 8. Autenticazione e autorizzazione ##
- Tutte le API servite su HTTPS.
- JWT con token bearer obbligatorio su tutte le API protette.
- Ruoli definiti: USER (clienti) e BACKOFFICE_OPERATOR.
- Controlli autenticazione e autorizzazione lato controller e service.
- Endpoint creazione ordine può essere pubblico con validazione idempotencyKey per sicurezza.
- Chiavi JWT firmate e validate secondo policy aziendale.

## 9. Gestione errori ##
- Risposte 400 Bad Request con dettagli per errori di validazione.
- Risposte 401/403 per errori di autenticazione/autorità.
- Risposte 500 con messaggi generici per errori interni, logging dettagliato per diagnostica.
- Retry automatici su chiamate a sistemi esterni con exponential backoff.
- Rollback ACID per transazioni fallite, compensazioni su stock e pagamento.
- Idempotenza su creazione ordine, rimborso e aggiornamenti per evitare duplicazioni.
- Dead-letter queue per messaggi RabbitMQ problematici.
- Logging strutturato con correlazione requestId.
- Masking dati sensibili e compliance GDPR in log e audit.

## 10. Strategia di test backend ##

| Nome Test                               | Tipo          | Cosa verifica                                  | Input                                         | Output atteso                                  |
|----------------------------------------|---------------|-----------------------------------------------|-----------------------------------------------|-----------------------------------------------|
| CreateOrder_ValidRequest_ShouldSucceed | Integration   | Creazione ordine con dati validi e idempotency| Request body POST /api/orders completo        | 201 Created con orderId e stato iniziale       |
| CreateOrder_DuplicateIdempotencyKey    | Unit          | Blocco duplicazione ordini con stessa idempotencyKey | Due richieste con stesso idempotencyKey       | Seconda richiesta restituisce ordine esistente |
| GetOrder_ValidJWT_ShouldReturnOrder    | Integration   | Recupero ordine autenticato con JWT valido     | JWT valido, orderId esistente                   | 200 OK con dati ordine                           |
| Backoffice_Refund_WithValidAdmin       | Integration   | Rimborso manuale da backoffice con ruolo corretto | Request POST /refund con idempotencyKey       | 200 OK con stato rimborso e timestamp           |
| PaymentModule_RetryOnTransientFailure  | Unit          | Retry su chiamate Stripe in caso di errori temporanei | Simulazione errore rete Stripe                  | Metodo Stripe chiamato più volte con backoff    |
| StockModule_ConcurrentUpdate_Conflict  | Integration   | Gestione lock e concorrenza aggiornamento stock| Due aggiornamenti concorrenti stock per stesso SKU | Nessuna perdita o errore concurrente           |
| RabbitMQConsumer_Idempotency           | Unit          | Processing idempotente dei messaggi RabbitMQ  | Messaggi duplicati o ri-inviati                  | Stato aggiornato una sola volta                   |
| AuthMiddleware_InvalidToken_Denied     | Unit          | Accesso negato con token JWT scaduto o mancante | Richiesta API con token non valido              | 401 Unauthorized                                 |
| NotificationModule_SendEmail_OnStatusChange | Integration | Invio email su cambio stato ordine             | Evento stato ordine pagato, spedito             | Email inviata e log evento presente              |

## 11. Rischi tecnici ##
- Conflitti di concorrenza alta su aggiornamento stock e stato ordine.
- Timeout e fallimenti prolungati in integrazione con Stripe e servizi logistici.
- Mancata gestione corretta di idempotencyKey con doppia fatturazione o operazioni duplicate.
- Crescita e gestione dimensioni audit log impattanti performance DB.
- Vulnerabilità di sicurezza JWT e gestione ruoli insufficienti.
- Complessità rollback multi-sistema e consistenza transazionale durante guasti.
- Scalabilità RabbitMQ con ordering e duplicazioni messaggi.
- Performance API sotto carico pesante (>500 ordini/minuto) con latenza <200ms.
- Gestione incomplete o errate delle notifiche push e opt-out utenti.
- Compliance GDPR incompleta per dati sensibili e logging.

## 12. Struttura file proposta ##

```
backend/
│
├── src/
│   ├── main/
│   │   ├── java/com/company/ecommerce/
│   │   │   ├── controller/
│   │   │   │   ├── OrderController.java          # Gestione API ordini pubbliche
│   │   │   │   ├── BackofficeOrderController.java# API backoffice
│   │   │   │   ├── InternalCallbackController.java # RabbitMQ callback endpoint
│   │   │   │
│   │   │   ├── service/
│   │   │   │   ├── OrderService.java              # Logica business ordini
│   │   │   │   ├── PaymentService.java            # Gestione Stripe e rimborsi
│   │   │   │   ├── ShipmentService.java           # Integrazione logistica
│   │   │   │   ├── StockService.java              # Gestione stock e lock
│   │   │   │   ├── NotificationService.java       # Email, push asincroni
│   │   │   │
│   │   │   ├── repository/
│   │   │   │   ├── OrderRepository.java           # CRUD + query orders
│   │   │   │   ├── PaymentRepository.java         # Pagamenti DB
│   │   │   │   ├── ShipmentRepository.java        # Spedizioni DB
│   │   │   │   ├── StockRepository.java           # Stock DB
│   │   │   │   ├── AuditLogRepository.java        # Audit log DB
│   │   │   │
│   │   │   ├── integration/
│   │   │   │   ├── StripeClient.java               # Wrapper Stripe API
│   │   │   │   ├── LogisticsClient.java            # Wrapper logistica
│   │   │   │   ├── SendGridClient.java             # Wrapper SendGrid
│   │   │   │   ├── RabbitMQConsumer.java           # Listener eventi RabbitMQ
│   │   │   │
│   │   │   ├── security/
│   │   │   │   ├── JwtAuthFilter.java              # Middleware JWT
│   │   │   │   ├── RoleAuthorization.java          # Controlli ruoli
│   │   │   │
│   │   │   ├── model/
│   │   │   │   ├── Order.java                       # Entity ordine
│   │   │   │   ├── Payment.java                     # Entity pagamento
│   │   │   │   ├── Shipment.java                    # Entity spedizione
│   │   │   │   ├── StockItem.java                   # Entity stock
│   │   │   │   ├── AuditLog.java                    # Entity audit log
│   │   │   │
│   │   │   ├── exception/
│   │   │   │   ├── CustomException.java            # Classi eccezioni personalizzate
│   │   │   │
│   │   ├── resources/
│   │   │   ├── application.properties              # Configurazioni Bear, DB, Stripe, RabbitMQ
│   │   │   ├── logback.xml                          # Configurazione logging
│   │   │   ├── db/
│   │   │   │   ├── migrations/                      # Migration SQL scripts
│   │   │
│   ├── test/
│   │   ├── java/com/company/ecommerce/              # Test unit e integration corrispondenti
│   │   │   ├── OrderServiceTest.java
│   │   │   ├── PaymentServiceTest.java
│   │   │   ├── RabbitMQConsumerTest.java
│   │   │   ├── AuthMiddlewareTest.java
│
├── Dockerfile                                        # Container setup
├── README.md                                         # Documentazione progetto
```

## 13. Piano di implementazione ##
1. Setup ambiente sviluppo, repository e configurazioni Bear e PostgreSQL.
2. Definizione modelli dati e migrazioni DB con script SQL.
3. Implementazione JWT middleware e controlli di sicurezza.
4. Sviluppo repository layer e test unitari CRUD.
5. Implementazione OrderService con flusso creazione ordine, idempotenza e validazioni.
6. Integrazione Stripe (autorizzazione e capture) con test mocking.
7. Implementazione moduli Shipment e Stock con locking e transazioni.
8. Configurazione RabbitMQ producer e consumer, gestione evento aggiornamento stato.
9. Sviluppo API Controller e rotte principali con validazioni.
10. Implementazione modulo refund backoffice con idempotency.
11. Implementazione Notification Module e invio email/push asincroni.
12. Testing end-to-end e integrazione completa, gestione errori e rollback.
13. Setup CI/CD pipeline con test automatizzati e deploy.
14. Monitoraggio performance e tuning DB, API e RabbitMQ.
15. Preparazione documentazione operativa e SLA.

---

Questa proposta backend consolidata garantisce un sistema robusto, scalabile e sicuro per il microservizio di gestione ordini e-commerce, conforme ai requisiti richiesti, integrando le migliori pratiche di sviluppo, testing, sicurezza e resilienza.
```