MODULE: BACKEND VERSION: 1

## 1. Obiettivo backend ##
Il backend è un microservizio worker progettato per la gestione completa degli ordini di un e-commerce. Gestisce il ciclo di vita degli ordini, inclusa la ricezione, validazione, pagamento, spedizione, notifiche, storicizzazione e rimborsi, garantendo consistenza transazionale, sicurezza, scalabilità e resilienza alle integrazioni esterne.

## 2. Assunzioni tecniche ##
- Backend basato su Java con Spring Boot.  
- PostgreSQL per persistenza dati con supporto ACID e locking a livello di riga.  
- RabbitMQ per la gestione asincrona degli eventi (event-driven).  
- Autenticazione via JWT Bearer Token con controlli di ruolo (clienti, operatori).  
- Integrazione con Stripe per pagamenti, servizio logistico per spedizioni e SendGrid per notifiche email.  
- Tutte le operazioni critiche sono transazionali con rollback o compensazione.  
- Input validati e sanificati per sicurezza da injection e attacchi.  
- Logging e audit immutabile con conformità GDPR.

## 3. Architettura backend ##
Architettura microservizio a livelli con modello event-driven interno:  
- **API Layer (Controller)**: Esponibile via REST JSON con endpoint autenticati.  
- **Service Layer**: Logica di dominio, gestione stati ordine, orchestrazione pagamenti, spedizioni e notifiche.  
- **Persistence Layer**: Repository dedicati che implementano operazioni transazionali su PostgreSQL.  
- **Integration Modules**: Interfaccia verso Stripe, servizi logistici, RabbitMQ e sistema notifiche.  
- **Async Event Consumer**: RabbitMQ consumer per aggiornamento stadi ordine in modo idempotente e ordinato.  
- **Security Module**: Gestisce autenticazione JWT e autorizzazione a livello API.  
- **Audit and Logging Module**: Traccia immutabile eventi e modifiche.

## 4. Moduli e responsabilità ##
- **OrderController**: gestione REST API per ordini clienti e backoffice.  
- **OrderService**: gestione business logic flussi ordine, stato, pagamenti e spedizioni.  
- **OrderRepository, PaymentRepository, InventoryRepository, ShipmentRepository, AuditLogRepository**: accesso e manipolazione dati persistenti.  
- **StripeIntegration**: chiamate autorizzazione e cattura pagamento con retry e idempotenza.  
- **LogisticsIntegration**: gestione creazione spedizioni e tracking con retry.  
- **NotificationModule**: invio di email e notifiche push tramite SendGrid e interfacce configurabili.  
- **RabbitMQConsumer**: ascolto e gestione eventi asincroni ordine con deduplicazione e gestione ordine messaggi.

## 5. API principali

### 5.1 POST /orders  
**Descrizione:** Crea nuovo ordine con payload validato.  
**Auth:** JWT utente cliente  
**Request:**  
```json
{
  "customer": {"id":"string","name":"string","email":"string","phone":"string"},
  "products": [{"productId":"string","quantity":integer}],
  "shippingAddress": {"street":"string","city":"string","postalCode":"string","country":"string"},
  "payment": {"method":"card","cardData":{"token":"string"},"amount":"decimal","currency":"string"}
}
```  
**Response:**  
- 200 OK  
```json
{"orderId":"string","status":"VALIDATION_PENDING","createdAt":"ISO8601 timestamp"}
```
- 400 Bad Request con dettagli errori  
- 500 Internal Server Error

### 5.2 GET /backoffice/orders  
**Descrizione:** Lista e filtra ordini per backoffice con paginazione.  
**Auth:** JWT ruolo operatore/admin  
**Query Params:** status, startDate, endDate, page, pageSize  
**Response 200 OK:**  
```json
{
  "orders": [{"orderId":"string","status":"string","customerName":"string","createdAt":"ISO8601 timestamp","totalAmount":"decimal","currency":"string"}],
  "pagination":{"currentPage":integer,"totalPages":integer,"pageSize":integer,"totalItems":integer}
}
```

### 5.3 GET /backoffice/orders/{orderId}  
**Descrizione:** Dettaglio ordine completo.  
**Auth:** JWT ruolo operatore/admin  
**Response 200 OK:**  
```json
{
  "orderId":"string","status":"string",
  "customer":{"id":"string","name":"string","email":"string","phone":"string"},
  "products":[{"productId":"string","name":"string","quantity":integer,"unitPrice":"decimal","totalPrice":"decimal"}],
  "shippingAddress":{"street":"string","city":"string","postalCode":"string","country":"string"},
  "paymentDetails":{"method":"card","authorizationId":"string","captureId":"string","amount":"decimal","currency":"string","status":"authorized|captured|failed"},
  "shipment":{"shipmentId":"string","trackingNumber":"string","status":"string","carrier":"string"},
  "createdAt":"ISO8601 timestamp","updatedAt":"ISO8601 timestamp"
}
```

### 5.4 POST /backoffice/orders/{orderId}/refund  
**Descrizione:** Esegue rimborso totale ordine.  
**Auth:** JWT ruolo operatore/admin  
**Request:** Corpo vuoto (solo rimborso totale)  
**Response:**  
- 200 OK  
```json
{"orderId":"string","refundStatus":"processed","refundedAt":"ISO8601 timestamp"}
```
- 400/409 con dettagli errori

## 6. Business logic ##
- Gestione stati ordine (VALIDATION_PENDING, AUTHORIZED, PAID, SHIPPED, DELIVERED, CANCELED, REFUNDED).  
- Autorizzazione pagamento con Stripe su creazione ordine; cattura asincrona, rollback su errori.  
- Locking e decremento atomico stock via DB in seguito a pagamento.  
- Creazione spedizione tramite modulo logistico con retry e idempotenza.  
- RabbitMQ per eventi stato ordine con processamento idempotente e ordinato.  
- Notifiche contestuali ad ogni cambio stato (email e push).  
- Rimborso integrato con rollback e aggiornamento stock.

## 7. Persistenza e integrazioni ##
- PostgreSQL schema per ordini, pagamenti, inventario, spedizioni, audit.  
- Repository JPA o ORM leggero con transazioni, locking, retry.  
- Integrazione Stripe SDK Java con retry, idempotenza e mapping errori.  
- Integrazione servizio logistico REST con circuit breaker e retry.  
- RabbitMQ consumer garantisce esecuzione affidabile e deduplicata.  
- Invio notifiche via SendGrid e moduli push configurabili.

## 8. Autenticazione e autorizzazione ##
- JWT mandatory su tutte le API.  
- Verifica firma, scadenza, ruoli utente.  
- Accesso backoffice limitato a operatori e admin.  
- Risposte HTTP 401/403 coerenti.  
- Nessuna fuga di dettagli su errori di sicurezza.

## 9. Gestione errori ##
- Validazioni payload risposte 400 con dettagli.  
- 401 per errori autenticazione, 403 per autorizzazione.  
- 404 se risorsa non trovata.  
- 409 per conflitti di business (es. rimborso impossibile).  
- 500 per errori server con messaggi generici.  
- Retry con backoff per chiamate esterne con idempotenza.  
- Logging strutturato e correlazione eventi per tracciabilità.

## 10. Strategia di test backend ##
- **CreateOrder_ValidPayload_Unit**: test unitario validazione e mapping payload ordine, input: payload valido, output: ordine creato con stato iniziale.  
- **CreateOrder_InvalidPayload_Unit**: test unitario verifica risposta 400 su dati errati.  
- **OrderService_PaymentAuthorization_Integration**: test integrazione simulata con Stripe, verifica autorizzazione pagamento e filtro errori.  
- **OrderRepository_Concurrency_Integration**: test concorrenza DB con locking su stock decrementi.  
- **RabbitMQConsumer_Idempotency_Integration**: verifica deduplicazione e ordine messaggi evento ordine.  
- **RefundOrder_Backoffice_E2E**: test end-to-end flusso rimborso via API backoffice con rollback stock e stato ordine.

## 11. Rischi tecnici ##
- Concorrenza e race condition su aggiornamenti stock e stato ordine.  
- Duplicazioni eventi RabbitMQ con possibili stati incoerenti.  
- Dipendenza da disponibilità servizi esterni Stripe e logistici.  
- Gestione ordini incompleti o payload inconsistenti.  
- Compliance GDPR su audit log immutabile e protezione dati.  
- Vulnerabilità sicurezza (injection, auth) mitigata da best practices.

## 12. Struttura file proposta ##
```
/src
  /api
    OrderController.java           # REST API endpoint e validazioni
  /service
    OrderService.java              # Logica dominio gestione ordini e flussi
  /repository
    OrderRepository.java           # CRUD ordini e locking transazionali
    PaymentRepository.java         # Gestione dati pagamenti e idempotenza
    InventoryRepository.java       # Controllo stock e locking
    ShipmentRepository.java        # Tracking creazione spedizioni
    AuditLogRepository.java        # Scrittura audit immutabile
  /integration
    StripeIntegration.java         # Integrazione pagamenti Stripe
    LogisticsIntegration.java      # Chiamate servizio spedizioni
    NotificationModule.java        # Invio notifiche email/push
  /async
    RabbitMQConsumer.java          # Consumatore eventi asincroni ordine
  /security
    JwtAuthenticationFilter.java  # Filtro JWT token e verifica ruoli
  /model
    Order.java                    # Entità dominio ordine
    Payment.java                  # Entità pagamento
    Inventory.java                # Entità stock/prodotto
    Shipment.java                 # Entità spedizione
    AuditLog.java                 # Entità audit log
  /config
    DatabaseConfig.java           # Config DB PostgreSQL, transazioni
    RabbitMQConfig.java           # Config RabbitMQ consumer/producer
    SecurityConfig.java           # Config sicurezza JWT e autorizzazioni
  /util
    RetryUtils.java               # Logica retry e backoff per integrazioni
```

## 13. Piano di implementazione ##
1. Setup ambiente di sviluppo, repository e configurazione database PostgreSQL.  
2. Implementazione modelli dominio e schema DB con tabelle ordini, pagamenti, inventari, spedizioni, audit.  
3. Sviluppo repository con supporto transazioni e locking.  
4. Implementazione API REST di base (OrderController) con validazione payload.  
5. Sviluppo OrderService per gestione flussi di stato e orchestrazione integrazioni.  
6. Integrazione modulo Stripe con gestione retry e mapping stato pagamento.  
7. Integrazione modulo logistico per creazione e tracking spedizioni.  
8. Realizzazione RabbitMQ consumer per gestione eventi asincroni con deduplicazione.  
9. Implementazione NotificationModule per invio email e push.  
10. Configurazione sicurezza JWT e controllo accessi per API.  
11. Implementazione error handling, logging e audit log immutabile.  
12. Sviluppo e automazione test unitari, integrazione ed end-to-end.  
13. Deployment in ambiente containerizzato con monitoring e health check.  
14. Revisione e ottimizzazione performance e scalabilità.  
15. Documentazione tecnica e handover.