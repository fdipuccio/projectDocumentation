MODULE: BACKEND VERSION: 1

## 1. Obiettivo backend
Realizzare un microservizio worker per la gestione ordini e-commerce che garantisca un workflow end-to-end completo: dalla creazione dell'ordine, validazione, gestione magazzino, processo di pagamento con Stripe, conferma ordine, integrazione spedizioni e gestione rimborsi. Il sistema deve essere robusto, sicuro, scalabile e conforme a GDPR, con comunicazioni sincrone via REST API e asincrone tramite RabbitMQ.

## 2. Assunzioni tecniche
- Backend sviluppato in Java usando un’architettura a microservizi.
- Persistenza dati con PostgreSQL con transazioni ACID.
- Comunicazioni esterne REST JSON con validazione JSON schema rigorosa.
- Autenticazione OAuth2 Bearer JWT obbligatoria su tutte le API protette.
- Messaging asincrono tramite RabbitMQ per eventi di stato ordine.
- Implementazione di pattern repository per accesso dati.
- Gestione idempotente e retry con backoff su chiamate esterne (Stripe, logistica, notifiche).
- Logging centralizzato e audit log per compliance GDPR.
- Locking pessimista per la gestione concorrenziale del magazzino.

## 3. Architettura backend
Microservizio strutturato secondo pattern layered con: 
- API REST sincrone per interazioni esterne.
- Business logic layer con orchestrazione workflow ordini.
- Repository layer per accesso isolato a database PostgreSQL.
- Consumer RabbitMQ per gestione eventi asincroni.
- Client REST per integrazioni esterne Stripe, logistica e notifiche.
- Sistema di logging e audit integrato.
- Meccanismi di autenticazione e autorizzazione integrati all'API Gateway o middleware.

## 4. Moduli e responsabilità
- Controller REST: espone endpoint per creazione, lettura, lista ordini e rimborso.
- Service Layer: orchestrazione logica ordini, pagamenti, transizioni di stato.
- Repository: gestione entità (Order, OrderItem, Payment, Inventory, Shipment, AuditLog, NotificationEvent).
- Integration Clients: Stripe Gateway, Logistics Service, Notification Service.
- RabbitMQ Consumer: ascolta eventi asincroni di stato e li gestisce garantendo idempotenza.
- Security Module: verifica JWT, applica RBAC.
- Error Handling Module: uniforma gestione errori e retry.
  
## 5. API principali

### 5.1 POST /api/orders
- Metodo: POST
- Path: /api/orders
- Request Schema:
```json
{
  "customer": {
    "id": "string (uuid)",
    "name": "string",
    "email": "string (email format)",
    "phone": "string (optional)"
  },
  "items": [
    {
      "productId": "string (uuid)",
      "quantity": "integer (positive)"
    }
  ],
  "shippingAddress": {
    "street": "string",
    "city": "string",
    "postalCode": "string",
    "country": "string (ISO 3166-1 alpha-2)"
  },
  "payment": {
    "method": "string (stripe_card, stripe_wallet)",
    "authorizationId": "string (optional)"
  }
}
```
- Response Schema (201 Created):
```json
{
  "orderId": "string (uuid)",
  "status": "VALIDATION_PENDING",
  "createdAt": "ISO8601 timestamp"
}
```
- Esempio Request:
```json
{
  "customer": {"id": "uuid-1234", "name": "Mario Rossi", "email": "mario@esempio.com"},
  "items": [{"productId": "uuid-prod-5678", "quantity": 2}],
  "shippingAddress": {"street": "Via Roma 1", "city": "Milano", "postalCode": "20100", "country": "IT"},
  "payment": {"method": "stripe_card"}
}
```
- Gestione errori: 400, 401, 409, 500

### 5.2 GET /api/orders/{orderId}
- Metodo: GET
- Path: /api/orders/{orderId}
- Response Schema (200 OK):
```json
{
  "orderId": "string",
  "customer": {...},
  "items": [...],
  "shippingAddress": {...},
  "payment": {
    "status": "AUTHORIZED | CAPTURED | FAILED | PENDING",
    "method": "string",
    "authorizationId": "string"
  },
  "orderStatus": "...",
  "tracking": {...},
  "timestamps": {...}
}
```
- Esempio: Dettaglio ordine completo

### 5.3 GET /api/orders
- Metodo: GET
- Path: /api/orders
- Query Parameters: status, customerId, page, size
- Response Schema (200 OK):
```json
{
  "page": 1,
  "size": 20,
  "totalElements": 100,
  "orders": [/* array di sintesi ordini */]
}
```

### 5.4 POST /api/orders/{orderId}/refund
- Metodo: POST
- Path: /api/orders/{orderId}/refund
- Request: nessun body
- Response Schema (200 OK):
```json
{
  "orderId": "string",
  "refundStatus": "INITIATED | COMPLETED | FAILED"
}
```

### 5.5 GET /api/health
- Metodo: GET
- Path: /api/health
- Nessuna autenticazione richiesta
- Response 200 OK con stato di salute del servizio

## 6. Business logic
- Creazione ordine: validazione payload, verifica stock con locking pessimista, persistenza ordine.
- Pagamento: autorizzazione e capture stripe asincroni, gestione stati pagamento.
- Spedizione: integrazione asincrona logistica con retry e idempotenza.
- Refund: gestione rimborso completo, aggiornamento stato ordine e ripristino stock.
- Eventi RabbitMQ: aggiornamento stato ordine, notifiche, gestione idempotente ed ordinata.
- Notifiche: invio email e push per ogni transizione di stato.

## 7. Persistenza e integrazioni
- Database PostgreSQL con schema relazionale normalizzato e supporto ad ACID.
- Tabelle: Order, OrderItem, Payment, Inventory, Shipment, AuditLog, NotificationEvent.
- Repositories Java per isolamento accesso dati, con transazioni gestite declarativamente.
- Integrazione con Stripe (pagamenti), servizio logistica esterno, sistema notifiche email/push.
- Messaging RabbitMQ per eventi ordine asincroni.

## 8. Autenticazione e autorizzazione
- OAuth2 Bearer JWT obbligatorio su tutte le API protette.
- JWT validato a gateway o middleware API.
- RBAC definito:
  - Client generici: creano ordini, leggono propri ordini.
  - Operatori backoffice: listano ordini, processano rimborsi.
- Input rigorosamente validati e sensitivi cifrati.
- Health endpoint esposto senza auth per monitoraggio.

## 9. Gestione errori
- Standard HTTP status e response JSON uniforme:
```json
{
  "errorCode": "string",
  "message": "string",
  "details": {}
}
```
- Codici: VALIDATION_ERROR, UNAUTHORIZED, FORBIDDEN, CONFLICT, NOT_FOUND, EXTERNAL_SERVICE_ERROR, INTERNAL_ERROR.
- Retry automatico con backoff per servizi esterni.
- Logging con contesto per diagnosi e audit.

## 10. Strategia di test backend

| Nome Test                        | Tipo       | Verifica                                     | Input                         | Output                        |
|---------------------------------|------------|----------------------------------------------|-------------------------------|-------------------------------|
| CreateOrder_ValidPayload         | Unit       | Validazione payload e chiamata a DB          | Payload ordine valido          | Stato VALIDATION_PENDING       |
| CreateOrder_InsufficientStock    | Integration| Locking pessimista e gestione errore stock   | Ordine con quantità magazzino insuff. | Errore 409 CONFLICT          |
| Payment_Authorization_Success    | Integration| Integrazione Stripe autorizzazione            | Dati pagamento validi          | Stato PAYMENT_PENDING          |
| Shipping_Create_Success          | Integration| Chiamata asincrona al servizio logistica     | Ordine confermato              | Stato SHIPPING_CREATED         |
| Refund_FullProcess_Complete      | E2E        | Processo completo rimborso e rollback stock   | Richiesta rimborso             | Stato REFUNDED                |
| Auth_JWT_Validation              | Unit       | Verifica token JWT e RBAC                      | Richiesta con/ senza JWT       | Accesso consentito / negato    |
| RabbitMQ_Event_Processing_Idempotent | Integration | Gestione eventi duplicati e fuori ordine      | Messaggi duplicati o ritardati | Stato consistente ordine      |

## 11. Rischi tecnici
- Overselling per lock contention su magazzino o errata gestione locking.
- Messaggi RabbitMQ duplicati o fuori ordine causando inconsistenze.
- Dipendenza da servizi esterni (Stripe, logistica) con possibili outage prolungati.
- Incompleta definizione casi rimborsi parziali e sistema notifiche push.
- Compliance GDPR su dati sensibili e audit log.
- Sicurezza e validazione input per prevenire attacchi injection.

## 12. Struttura file proposta

```
/src
├── /controller
│     └── OrderController.java          # Espone REST API ordini
├── /service
│     ├── OrderService.java             # Logica business ordine
│     ├── PaymentService.java           # Gestione pagamenti Stripe
│     ├── ShippingService.java          # Integrazione logistica
│     └── RefundService.java            # Gestione rimborsi
├── /repository
│     ├── OrderRepository.java          # CRUD ordine
│     ├── PaymentRepository.java        # CRUD pagamento
│     ├── InventoryRepository.java      # Gestione magazzino e locking
│     ├── ShipmentRepository.java       # Tracking spedizioni
│     ├── AuditLogRepository.java       # Log audit compliance
│     └── NotificationEventRepository.java # Log notifiche inviate
├── /integration
│     ├── StripeClient.java             # Client REST Stripe
│     ├── LogisticsClient.java          # Client REST logistica
│     └── NotificationClient.java       # Client notifiche email/push
├── /consumer
│     └── OrderEventConsumer.java       # Consumer RabbitMQ eventi ordine
├── /security
│     ├── JwtFilter.java                # Filtro validazione JWT
│     └── RbacAuthorization.java       # Logica RBAC
├── /exception
│     └── ApiExceptionHandler.java      # Gestione centralizzata errori
└── /config
      └── AppConfig.java                # Config generali e DI
```

## 13. Piano di implementazione
1. Setup progetto e infrastruttura (DB, RabbitMQ, config sicurezza).
2. Implementazione modello dati e repository con transazioni e locking.
3. Sviluppo API REST di base (creazione e dettaglio ordine).
4. Integrazione con Stripe (authorization, capture) con retry e idempotenza.
5. Implementazione consumer RabbitMQ per gestione eventi.
6. Sviluppo processi spedizione e notifiche asincrone.
7. Implementazione endpoint rimborso e logica cancellazione stock.
8. Completamento sicurezza con JWT e RBAC.
9. Scrittura test unitari, di integrazione ed E2E.
10. Deploy ambiente test, esecuzione test e fix.
11. Documentazione tecnica e preparazione per QA.

---

Documento backend finale conforme ai requisiti tecnici, architetturali e di sicurezza, pronto per il ciclo di QA e produzione.