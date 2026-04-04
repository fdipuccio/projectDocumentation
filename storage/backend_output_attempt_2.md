MODULE: BACKEND VERSION: 1

## 1. Obiettivo backend
Implementare un microservizio worker per la gestione degli ordini e-commerce che esegua il flusso completo di ricezione, validazione, pagamento, gestione magazzino, integrazione logistica, notifiche e rimborso, garantendo coerenza, sicurezza, performance e scalabilità secondo le specifiche architetturali fornite.

## 2. Assunzioni tecniche
- Backend worker basato su Java con Bear Framework.
- Database PostgreSQL garantendo transazioni ACID.
- Comunicazioni sincrone REST API e asincrone via RabbitMQ.
- Sicurezza tramite JWT Bearer Token.
- Integrazioni esterne con Stripe, servizi logistici e SendGrid.
- Pattern architetturale: microservizi, event-driven, layered.
- Logging e audit GDPR-compliant.
- Controllo idempotente per flussi critici.
- Deploy containerizzato, scalabilità orizzontale.

## 3. Architettura backend
Architettura microservizi con componenti containerizzati autonomi. Il backend espone API REST protette con JWT e utilizza RabbitMQ come bus eventi per aggiornamenti stato ordine e integrazione asincrona. La persistenza è gestita da PostgreSQL con locking pessimista per la gestione stock. L’architettura prevede moduli interni per integrazioni esterne (Stripe, logistica, notifiche), logging centralizzato e gestione errori con retry e idempotenza. Transazioni ACID garantiscono coerenza dati.

## 4. Moduli e responsabilità
- API Gateway/Controller: espone endpoint REST, valida input, autorizza utenti.
- Order Service: logica business ordine, gestione stati, workflow.
- Payment Module: gestione pagamento (authorization, capture) via Stripe.
- Stock Module: gestione scorte con locking pessimista.
- Shipping Module: integrazione con servizio logistico esterno.
- Notification Module: invio email/push notifications.
- RabbitMQ Consumer: processa eventi asincroni, garantisce idempotenza e ordine.
- Persistence Layer: repository ORM con transazioni per entità (Order, Payment, Stock, Shipment, AuditLog, NotificationLog).
- Error Handling & Logging: gestisce errori, retry, auditing.

## 5. API principali

### POST /api/orders
- Descrizione: Crea un nuovo ordine e avvia workflow.
- Autenticazione: JWT cliente obbligatorio.
- Request schema:
```json
{
  "customer": {"id": "string","name": "string","email": "string","phone": "string"},
  "products": [{"productId": "string","quantity": "integer"}],
  "shippingAddress": {"street": "string","city": "string","postalCode": "string","country": "string"},
  "payment": {
    "method": "card",
    "cardDetails": {"cardNumber": "string","expiryDate": "string","cvv": "string","holderName": "string"},
    "authorizationId": "string (optional)"
  }
}
```
- Response 201:
```json
{
  "orderId": "UUID",
  "status": "VALIDATED",
  "createdAt": "ISO8601 timestamp"
}
```
- Errori: 400, 401, 500.

### GET /api/orders
- Descrizione: Lista ordini per operatori backoffice con filtri su stato e data, paginazione.
- Autenticazione: JWT operatore.
- Response 200:
```json
{
  "orders": [
    {"orderId": "UUID","customerName": "string","status": "string","createdAt": "ISO8601 timestamp","totalAmount": "decimal"}
  ],
  "pagination": {"page": "integer","pageSize": "integer","totalPages": "integer","totalRecords": "integer"}
}
```

### GET /api/orders/{orderId}
- Descrizione: Dettaglio ordine specifico.
- Autenticazione: JWT operatore.
- Response 200:
```json
{
  "orderId": "UUID",
  "customer": {"id": "string","name": "string","email": "string","phone": "string"},
  "products": [{"productId": "string","quantity": "integer","price": "decimal"}],
  "shippingAddress": {"street": "string","city": "string","postalCode": "string","country": "string"},
  "payment": {"status": "AUTHORIZED | CAPTURED | FAILED","authorizationId": "string","captureId": "string (optional)"},
  "status": "string",
  "createdAt": "ISO8601 timestamp",
  "updatedAt": "ISO8601 timestamp",
  "tracking": {"shipmentId": "string","status": "string","trackingUrl": "string"}
}
```

### POST /api/orders/{orderId}/refund
- Descrizione: Rimborso totale manuale ordine.
- Autenticazione: JWT operatore.
- Request body: vuoto `{}`.
- Response 200:
```json
{
  "orderId": "UUID",
  "refundStatus": "COMPLETED | FAILED",
  "refundTimestamp": "ISO8601 timestamp"
}
```

## 6. Business logic
- Ricezione ordine con validazioni e persistenza in stato VALIDATED.
- Evento RabbitMQ genera workflow asincrono per pagamento.
- Pagamento stripe con flusso authorization e capture separato, retry e idempotenza.
- Gestione stock con locking pessimista per blocco/release quantità.
- Workflow stato ordine con transizioni controllate e evento di stato su ogni cambiamento.
- Integrazione shipping con gestione asincrona tracking.
- Notifiche email/push per cambi stato ordine.
- Rimborso totale manuale aggiorna stato, ripristina stock, e registra audit.

## 7. Persistenza e integrazioni
- PostgreSQL con entity Order, OrderItem, Payment, Stock, Shipment, AuditLog, NotificationLog.
- Transazioni ACID per operazioni critiche e coerenza.
- Repository pattern con ORM (JPA/Hibernate su Bear).
- Integrazioni con Stripe (pagamenti), Logistics API (spedizioni), SendGrid (email).
- RabbitMQ per eventi stato ordine con consumer idempotente e sequenziato.
- Retry automatici e circuit breaker per chiamate esterne.

## 8. Autenticazione e autorizzazione
- JWT Bearer Token per tutte le API REST.
- Ruoli: cliente per creazione ordine, operatore backoffice per gestione ordini.
- Validazione ruoli nei controller per accesso differenziato.
- HTTPS obbligatorio.
- Sanificazione degli input per evitare injection.
- Protezione dati sensibili, crittografia at-rest per dati sensibili.

## 9. Gestione errori
- Standard HTTP error codes:
  - 400 Bad Request per input non validi.
  - 401 Unauthorized per mancanza o invalidità token.
  - 403 Forbidden per permessi negati.
  - 404 Not Found per ordini non esistenti.
  - 409 Conflict per stati ordine non coerenti.
  - 500 Internal Server Error per errori imprevisti.
- Messaggi di errore chiari con campo error.
- Retry con exponential backoff per errori temporanei nelle chiamate esterne.
- Idempotenza per evitare duplicazioni.
- Logging centralizzato con informazioni di contesto e stack trace.
- Audit log per tutte le modifiche critiche e operazioni.

## 10. Strategia di test backend

| Nome Test                      | Tipo        | Cosa verifica                                           | Input                  | Output Atteso                          |
|-------------------------------|-------------|--------------------------------------------------------|------------------------|--------------------------------------|
| OrderService_CreateValidOrder  | Unit        | Validazione corretta e persistenza nuovo ordine.       | Payload ordine valido   | Stato ordine VALIDATED, ID generato  |
| OrderService_CreateOrder_InvalidInput | Unit | Gestione input malformato e validazione errori.        | Payload mancante campi  | Errore 400 Bad Request                |
| PaymentModule_AuthorizePayment | Integration | Flusso pagamento authorization Stripe con retry.       | Dati pagamento validi  | Stato pagamento AUTHORIZED            |
| PaymentModule_CapturePayment   | Integration | Flusso capture con gestione errori e retry.            | Pagamento autorizzato  | Stato pagamento CAPTURED o FAILED    |
| StockRepository_ManageStock    | Integration | Locking pessimista e aggiornamento quantità stock.     | Richiesta decremento   | Stock aggiornato correttamente       |
| OrderApi_GetOrderDetail        | E2E         | Endpoint GET /api/orders/{orderId} risponde dati completi| JWT operatore valido   | JSON dettaglio ordine 200 OK         |
| OrderApi_PostRefund_Success    | E2E         | Rimborso totale via API con aggiornamento stato.       | OrderId valido, JWT op | Refund COMPLETED, stato REFUNDED     |

## 11. Rischi tecnici
- Contesa concorrenziale per gestione stock in situazioni di scorte esigue.
- Gestione complessa di messaggi RabbitMQ duplicati o fuori sequenza.
- Dipendenza da servizi esterni (Stripe, logistica) con possibili downtime.
- Consistenza dati su flussi multi-step (pagamento, spedizione) con fallimenti parziali.
- Compliance GDPR su audit log e crittografia dati.
- Rischi di sicurezza: validazione input e gestione JWT.
- Ambiguità requisito su pagamenti parziali e regole rimborso.
- Complessità nelle notifiche push da definire.

## 12. Struttura file proposta
```
backend-order-service/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   ├── com/ecommerce/order/
│   │   │   │   ├── controller/
│   │   │   │   │   ├── OrderController.java          # gestione endpoint REST
│   │   │   │   ├── service/
│   │   │   │   │   ├── OrderService.java             # logica business ordine
│   │   │   │   │   ├── PaymentService.java           # gestione integrazione Stripe
│   │   │   │   │   ├── StockService.java             # gestione stock e locking
│   │   │   │   │   ├── ShippingService.java          # integrazione logistica
│   │   │   │   │   ├── NotificationService.java      # invio notifiche email/push
│   │   │   │   ├── repository/
│   │   │   │   │   ├── OrderRepository.java          # accesso datos persistence ordine
│   │   │   │   │   ├── PaymentRepository.java
│   │   │   │   │   ├── StockRepository.java
│   │   │   │   │   ├── ShipmentRepository.java
│   │   │   │   │   ├── AuditLogRepository.java
│   │   │   │   │   ├── NotificationLogRepository.java
│   │   │   │   ├── messaging/
│   │   │   │   │   ├── OrderStatusConsumer.java       # consumer RabbitMQ per eventi stato
│   │   │   │   ├── model/
│   │   │   │   │   ├── Order.java                     # entity ordine e correlati
│   │   │   │   │   ├── Payment.java
│   │   │   │   │   ├── Stock.java
│   │   │   │   │   ├── Shipment.java
│   │   │   │   │   ├── AuditLog.java
│   │   │   │   │   ├── NotificationLog.java
│   │   ├── resources/
│   │   │   ├── application.properties                # configurazioni
│   │   │   ├── logback.xml                            # configurazione logging
├── test/
│   ├── unit/
│   │   ├── service/
│   │   │   ├── OrderServiceTest.java
│   │   │   ├── PaymentServiceTest.java
│   ├── integration/
│   │   ├── repository/
│   │   │   ├── OrderRepositoryTest.java
│   ├── e2e/
│   │   ├── OrderApiE2ETest.java
├── Dockerfile
├── README.md
└── build.gradle
```

## 13. Piano di implementazione
1. Setup progetto Bear Framework, configurazione PostgreSQL e RabbitMQ.
2. Implementazione modelli entità e repository per persistenza.
3. Sviluppo API Controller e validazione input con security JWT.
4. Realizzazione logica business core in OrderService incluso workflow stati.
5. Integrazione Stripe con retry, idempotenza, flusso authorization + capture.
6. Implementazione gestione stock con locking pessimista.
7. Setup consumer RabbitMQ per eventi stato ordine.
8. Implementazione modulo integrazione logistica con retry e idempotenza.
9. Sviluppo NotificationService per email e push notifications.
10. Realizzazione endpoint rimborso manuale con aggiornamento stato e audit.
11. Logging centralizzato e audit compliance GDPR.
12. Implementazione test unitari, integration e E2E complete.
13. Deployment containerizzato con scalabilità configurata.
14. Test di performance e tuning di locking/concorrenza.
15. Documentazione finale API e manuale operativo.

---

Tale proposta backend è coerente con i requisiti, integra i contributi forniti eliminando duplicazioni e incongruenze, ed è pronta per il ciclo di QA e sviluppo.