```markdown
MODULE: BACKEND VERSION: 1

## 1. Obiettivo backend

Progettare e implementare un microservizio backend di tipo worker per la gestione ordini e-commerce con i seguenti obiettivi:
- Orchestrare flusso ordine end-to-end: creazione, validazione, pagamento, stock, spedizione.
- Garantire idempotenza, resilienza, compliance GDPR e PCI-DSS, logging audit immutabile.
- Esporre API REST sicure per clienti e backoffice con controllo accessi e gestione ruoli.
- Integrare con servizi esterni Stripe (pagamenti), Logistica, SendGrid (email).
- Gestire aggiornamenti asincroni stato ordine da RabbitMQ con deduplicazione e ordering.
- Mantenere performance elevate e transazioni ACID sul database PostgreSQL.

## 2. Assunzioni tecniche

- Linguaggio e framework: Java con Spring Boot / Spring Data JPA.
- Database relazionale PostgreSQL con transazioni ACID.
- Architettura microservizio event-driven layered.
- Messaggistica RabbitMQ per eventi asincroni.
- Integrazioni Stripe, Logistics, SendGrid tramite client SDK e REST.
- Autenticazione JWT Bearer Token per API REST.
- Idempotenza implementata con chiavi univoche (es. Idempotency-Key header).
- Logging strutturato JSON e audit log append-only immutabile.
- Retry automatici con backoff esponenziale e circuit breaker.
- Monitoraggio latenza API < 200 ms 95° percentile.

## 3. Architettura backend

- **API REST Layer:** Espone endpoint per ordini e backoffice, con sicurezza e validazione.
- **Application Layer:** Orchestrazione business logic; gestione stati ordine e workflow.
- **Integration Layer:** Client resiliente per Stripe (pagamenti), Logistica (spedizioni), SendGrid (email).
- **Async Event Consumer:** Processa eventi RabbitMQ OrderStatusUpdated con ordering e deduplicazione.
- **Persistence Layer:** PostgreSQL con schemi normalizzati per ordini, stock e audit log.

Tutti i layer rispettano il principio di separazione delle responsabilità; le operazioni critiche sono transazionali e garantiscono idempotenza e consistenza.

## 4. Moduli e responsabilità

| Modulo                  | Responsabilità principale                                                  |
|------------------------|---------------------------------------------------------------------------|
| API REST Controller     | Espone endpoint API, autenticazione JWT, validazione input, autorizzazioni|
| Order Service           | Gestione workflow ordine, stato, logica di business                       |
| Payment Integration     | Invocazioni Stripe authorize e capture con retry e idempotenza            |
| Shipping Integration    | Creazione spedizioni con servizio logistico, gestione tracking            |
| Email Notification      | Invio email via SendGrid su eventi stato ordine                           |
| Stock Management        | Controllo e aggiornamento stock magazzino con transazioni atomiche        |
| Event Consumer          | Consumo eventi RabbitMQ, deduplicazione, ordering, aggiornamento stato    |
| Persistence Repository  | Accesso dati su PostgreSQL via ORM, transazioni, locking                  |
| Audit Log Service       | Registrazione log immutabili per tutte le transizioni e azioni critiche   |

## 5. API principali

### 1. POST /api/orders - Creazione nuovo ordine

- Metodo: POST
- Path: /api/orders
- Headers:
  - Authorization: Bearer <jwt-token>
  - Content-Type: application/json
  - Idempotency-Key: <string-univoco>
- Request Body:
```json
{
  "customerId": "uuid",
  "items": [
    {
      "productId": "uuid",
      "quantity": 2
    }
  ],
  "payment": {
    "method": "card",
    "paymentInfo": {
      "token": "stripe_payment_token"
    }
  },
  "shippingAddress": {
    "name": "Nome Cliente",
    "addressLine1": "Via Esempio 123",
    "city": "Milano",
    "postalCode": "20100",
    "country": "IT"
  }
}
```
- Response 201 Created:
```json
{
  "orderId": "uuid",
  "status": "VALIDATION_PENDING",
  "createdAt": "ISO8601 timestamp",
  "totalAmount": 123.45,
  "message": "Ordine ricevuto e in elaborazione"
}
```
- Errori:
  - 400 Bad Request: Validazione input fallita.
  - 401 Unauthorized: JWT mancante o non valido.
  - 409 Conflict: Duplicato Idempotency-Key.
  - 500 Internal Server Error.

---

### 2. GET /api/orders/{orderId} - Dettaglio ordine cliente

- Metodo: GET
- Path: /api/orders/{orderId}
- Headers:
  - Authorization: Bearer <jwt-token>
- Response 200 OK:
```json
{
  "orderId": "uuid",
  "customerId": "uuid",
  "items": [
    {
      "productId": "uuid",
      "quantity": 2,
      "unitPrice": 50.00
    }
  ],
  "status": "CONFIRMED",
  "totalAmount": 100.00,
  "paymentStatus": "CAPTURED",
  "shippingStatus": "IN_TRANSIT",
  "shippingTracking": {
    "trackingId": "string",
    "carrier": "CarrierName",
    "estimatedDelivery": "ISO8601"
  },
  "createdAt": "ISO8601 timestamp",
  "updatedAt": "ISO8601 timestamp"
}
```

---

### 3. GET /api/backoffice/orders - Lista ordini backoffice

- Metodo: GET
- Path: /api/backoffice/orders
- Query params: page, size, status (opzionali)
- Authorization: JWT con ruolo amministratore
- Response 200 OK:
```json
{
  "orders": [
    {
      "orderId": "uuid",
      "status": "SHIPPED",
      "totalAmount": 123.45,
      "createdAt": "ISO8601",
      "customerId": "uuid"
    }
  ],
  "pagination": {
    "page": 1,
    "size": 20,
    "totalPages": 10,
    "totalElements": 200
  }
}
```

---

### 4. GET /api/backoffice/orders/{orderId} - Dettaglio ordine backoffice

- Metodo: GET
- Path: /api/backoffice/orders/{orderId}
- Authorization: JWT con ruolo amministratore
- Response simile al GET cliente ma comprensivo di dati amministrativi.

---

### 5. POST /api/backoffice/orders/{orderId}/refund - Rimborso manuale

- Metodo: POST
- Path: /api/backoffice/orders/{orderId}/refund
- Headers: Authorization JWT admin
- Body:
```json
{
  "reason": "Motivo rimborso descrittivo"
}
```
- Response 200 OK:
```json
{
  "orderId": "uuid",
  "refundStatus": "PROCESSING",
  "message": "Rimborso avviato"
}
```
- Errori:
  - 400 Bad Request: ordine non rimborsabile.
  - 403 Forbidden: ruolo non autorizzato.
  - 500 Internal Server Error.

## 6. Business logic

- Flusso ordine a stati finiti: VALIDATION_PENDING → PAYMENT_AUTHORIZED → PAYMENT_CAPTURED → CONFIRMED → SHIPPED → DELIVERED.
- Idempotenza garantita su operazioni critiche tramite header `Idempotency-Key`.
- Gestione pagamenti autorizzazione e capture con Stripe, retry con backoff esponenziale.
- Scalatura e ripristino stock in transazioni ACID.
- Creazione spedizioni con retry e idempotenza.
- Invio email cliente in ogni cambio stato.
- Consumo eventi RabbitMQ OrderStatusUpdated asincrono con deduplicazione e ordering garantito.
- Rimborso manuale gestito da backoffice con rollback e aggiornamento stock.
- Audit log immutabile per ogni azione di stato o mutazione dati.

## 7. Persistenza e integrazioni

- PostgreSQL con schema ordini, stock, audit log normalizzato.
- Repository Java Spring Data JPA per ogni entità.
- Transazioni DB per operazioni composte (es. pagamento + scalatura stock + audit).
- Integrazione Stripe con client idempotente e retry automatico.
- Integrazione Logistica per spedizioni idem.
- Integrazione SendGrid per notifiche email con retry e gestione errori.
- Event consumer per RabbitMQ con deduplicazione eventi tramite chiavi univoche e ordering.

## 8. Autenticazione e autorizzazione

- Tutti gli endpoint REST protetti da JWT Bearer Token.
- Validazione firma, scadenza e ruoli via filter/interceptor.
- Accesso cliente limitato ai propri ordini (verifica customerId).
- Accesso backoffice riservato a utenti con ruolo "amministratore" tramite sistema identity esterno.
- Nessun sistema di utenti interno al backend.
- Controlli di autorizzazione implementati rigidi sui ruoli.

## 9. Gestione errori

- Errori client: 400 con dettagli JSON su campi errati.
- Errori autenticazione/authorization: 401/403 con messaggi chiari.
- Idempotenza: richieste duplicate con stessa `Idempotency-Key` restituiscono successo con dati originari.
- Errori integrazione esterne gestiti con retry e failure escalation a 500.
- Event consumer ritenta eventi in caso di errori transitori.
- Transazioni DB rollback completo per garantire consistenza.
- Logging strutturato dettagliato per tracing e audit.
- Timeout e rate limiting monitorati e gestiti per mantenere performance.

## 10. Strategia di test backend

| Nome Test                                | Tipo          | Cosa verifica                                   | Input                                             | Output atteso                                      |
|-----------------------------------------|---------------|------------------------------------------------|--------------------------------------------------|---------------------------------------------------|
| OrderCreation_ValidInput                 | Unit          | Creazione ordine con input valido               | Request JSON corretto, JWT valido, Idempotency-Key | 201 Created + ordine creato con stato VALIDATION_PENDING |
| OrderCreation_DuplicateIdempotencyKey   | Integration   | Idempotenza su richieste duplicate              | Due richieste identiche con stesso Idempotency-Key | Seconda risposta 409 Conflict o 200 con dati originali |
| PaymentAuthorizationRetry                | Integration   | Retry autorizzazione pagamento Stripe           | Simulazione fallimento transitorio Stripe        | Successo dopo retry, stato ordine aggiornato       |
| StockScaling_TransactionalConsistency   | Unit/Integration | Scalatura stock atomica a pagamento confermato | Input ordine + pagamento catturato                | Stock aggiornato, transazione committata           |
| RefundManual_BackofficeAdmin             | Integration   | Rimborso manuale con controllo ruoli            | POST refund con token admin e ordine valido       | 200 OK, ordine REFUNDED, stock ripristinato        |
| EventConsumer_Deduplication              | Unit          | Deduplicazione e ordering eventi RabbitMQ       | Eventi duplicati e fuori ordine                    | Evento processato solo una volta in ordine corretto |
| AuthMiddleware_InvalidJWT                | Unit          | Blocco chiamata API con token JWT non valido    | Token scaduto o mancante                           | 401 Unauthorized                                    |
| APIResponse_ErrorSchema                  | Unit          | Response error coerenti e dettagliate            | Request malformata                                 | 400 Bad Request con payload di errore dettagliato  |

## 11. Rischi tecnici

1. Definizione event format e ordering RabbitMQ non completamente stabilita.
2. Rischio race condition su stato ordine e stock con concorrenza elevata.
3. Parametrizzazione retry e backoff da calibrare accuratamente per resilienza senza sovraccarico.
4. Gestione transazioni multi-step in presenza di errori (es: fallback capture, cancellazioni).
5. Implementazione compliance GDPR e PCI-DSS complessa.
6. Assicurare immutabilità e retention log audit persistenti e sicuri.
7. Integrazione role management backoffice da identity esterno, necessità di politiche least privilege rigorose.

## 12. Struttura file proposta

```
/src
  /main
    /java
      /com/company/order
        Application.java               # Classe main Spring Boot
        /api
          OrderController.java        # Espone API ordini cliente
          BackofficeController.java   # API backoffice con controlli ruolo
          AuthFilter.java             # Filtra JWT, verifica ruoli
        /service
          OrderService.java           # Business logic flusso ordine
          PaymentService.java         # Wrapper integrazione Stripe
          ShippingService.java        # Client Logistica spedizioni
          EmailNotificationService.java # Invio email SendGrid
          StockService.java           # Gestione stock magazzino
          RefundService.java          # Rimborso manuale backoffice
          EventConsumerService.java   # Consumatore eventi RabbitMQ
          AuditLogService.java        # Gestione audit log immutabili
        /repository
          OrderRepository.java        # Repository Spring Data JPA Orders
          StockRepository.java        # Repository stock magazzino
          AuditLogRepository.java     # Repository audit log
        /model
          Order.java                 # Modello dati ordine
          OrderItem.java
          Stock.java
          AuditLog.java
          PaymentInfo.java
          ShippingInfo.java
        /config
          AppConfig.java             # Configurazione retry, DB, JWT, RabbitMQ
        /exception
          ApiExceptionHandler.java   # Gestione centralizzata errori API
          CustomExceptions.java      # Definizione eccezioni custom
    /resources
      application.yml                # Configurazioni app (DB, servizi esterni)
      schema.sql                    # Script iniziale DB
      /migrations
        V1__init_schema.sql
  /test
    /java/com/company/order
      OrderServiceTest.java         # Unit test business logic ordine
      PaymentServiceTest.java
      StockServiceTest.java
      RefundServiceTest.java
      EventConsumerTest.java
      AuthFilterTest.java
      ApiExceptionHandlerTest.java
```

## 13. Piano di implementazione

1. Configurare ambiente di sviluppo: Java, Spring Boot, PostgreSQL, RabbitMQ.
2. Definire schema DB e implementare repository con Spring Data JPA.
3. Realizzare modulo API REST con autenticazione JWT e validazione input.
4. Implementare business logic ordinaria: creazione ordine, stato workflow, persistente su DB.
5. Integrare Stripe Payment: autorizzazione e capture con retry e idempotenza.
6. Implementare stock management transazionale.
7. Integrare Logistica spedizioni con client idempotente.
8. Realizzare servizio email SendGrid con retry.
9. Implementare consumer RabbitMQ per aggiornamenti asincroni.
10. Sviluppare moduli backoffice per rimborso manuale con controlli ruolo.
11. Aggiungere audit log immutabile e logging strutturato.
12. Implementare gestione errori centralizzata e messaggi coerenti.
13. Stesura test unitari e integration test su ogni modulo critico.
14. Deployment e test di carico per garantire performance target (<200ms 95° percentile).
15. Finalizzare documentazione e preparare ambiente monitoraggio e alerting.

---

La presente proposta backend unifica le specifiche funzionali, architetturali, di sicurezza, integrazione e performance nel rispetto dei requisiti PM. Il modello è progettato per garantire robustezza, scalabilità, manutenibilità e compliance normativa.

```