MODULE: BACKEND VERSION: 1

## 1. Obiettivo backend 
Sviluppare un microservizio backend worker per la gestione ordini e-commerce che garantisca un flusso end-to-end di creazione, gestione, pagamento, spedizione, rimborso e cancellazione ordini in modo scalabile, sicuro, resiliente e con completa tracciabilità. Il backend deve assicurare transazioni ACID, integrazione affidabile con servizi esterni (Stripe, logistica, notifiche), gestione asincrona tramite RabbitMQ e audit logging per compliance GDPR.

## 2. Assunzioni tecniche 
- Tecnologia Java con framework Bear e JPA/Hibernate.
- PostgreSQL come database relazionale con schema normalizzato e transazioni a isolamento serializzabile o Repeatable Read.
- Comunicazioni API REST sicure via HTTPS con autenticazione/autorizzazione JWT.
- Messaging asincrono basato su RabbitMQ con meccanismi di idempotenza e dead letter.
- Integrazioni esterne via HTTPS con autenticazioni dedicate (Stripe, logistica, SendGrid).
- Idempotenza garantita tramite token univoci per operazioni critiche.
- Logging strutturato JSON e audit logging immutabile per monitoraggio e diagnosi.
- Adozione di retry con backoff esponenziale e circuit breaker per errori temporanei.
- Gestione opt-out per notifiche push/email.

## 3. Architettura backend 
Architettura a microservizi con pattern event-driven:
- API Gateway che gestisce autenticazione JWT e autorizzazioni ruoli.
- Order Management Service come core service REST per ordini, pagamenti, eventi.
- Backoffice API per gestione ordini e rimborsi da operatori autenticati.
- Messaging RabbitMQ per eventi asincroni e aggiornamento stato ordine.
- Module per integrazione Stripe, Logistics, Notification.
- Persistence layer con repository pattern e transazioni coordinate.
- Meccanismi di cache per dati statici e ottimizzazioni performance.

## 4. Moduli e responsabilità 
- **API Gateway:** espone REST API, valida JWT, gestisce ruoli.
- **Order Management Service:** logica business ordini, pagamenti, stock, spedizioni, notifiche.
- **Backoffice API:** gestione ordini da operatori, richieste rimborso.
- **Stripe Integration Module:** comunicazione con API Stripe per payment lifecycle.
- **Logistics Integration Module:** creazione spedizioni e tracking.
- **Notification Module:** invio email/push, gestione opt-out.
- **Persistence Layer:** repository per ordini, pagamenti, stock, audit log.
- **Messaging Layer:** publisher/consumer RabbitMQ con idempotenza e retry.
- **Audit Logging:** registrazioni immutabili e conservazione per compliance.

## 5. API principali 

### Creazione Ordine
- Metodo: POST
- Path: /api/orders
- Request JSON schema:
```json
{
  "client_token": "string",
  "customer": {
    "name": "string",
    "email": "string",
    "phone": "string"
  },
  "items": [{"product_id": "string", "quantity": integer}],
  "shipping_address": {
    "street": "string",
    "city": "string",
    "postal_code": "string",
    "country": "string"
  },
  "payment_method": "stripe",
  "metadata": {"key": "value"}
}
```
- Response 201 Created:
```json
{
  "order_id": "uuid",
  "status": "CREATED",
  "created_at": "iso8601-timestamp"
}
```
- Esempio: Cliente crea ordine con token idempotenza, elenco prodotti e indirizzo.

---

### Recupero Dettaglio Ordine 
- Metodo: GET
- Path: /api/orders/{orderId}
- Response 200 OK con dettagli ordine, stato, pagamenti, spedizioni.

---

### Cancellazione Ordine 
- Metodo: POST
- Path: /api/orders/{orderId}/cancel
- Request:
```json
{
  "cancel_token": "string"
}
```
- Response 200 OK conferma cancellazione o errore stato non cancellabile.

---

### Autorizzazione Pagamento 
- Metodo: POST
- Path: /api/orders/{orderId}/payment/authorize
- Request:
```json
{
  "payment_token": "string",
  "amount": integer,
  "currency": "string",
  "payment_method_id": "string"
}
```
- Response 200 OK con stato autorizzazione e id pagamento.

---

### Cattura Pagamento 
- Metodo: POST
- Path: /api/orders/{orderId}/payment/capture
- Request con capture_token, autorizzazione id.

---

### Rimborso Manuale 
- Metodo: POST
- Path: /api/orders/{orderId}/payment/refund
- Richiesta idempotente con refund_token.

---

### Backoffice - Elenco ordini 
- Metodo: GET
- Path: /api/backoffice/orders
- Filtri e paginazione, accesso limitato a operatori.

---

### Backoffice - Dettaglio Ordine 
- Metodo: GET
- Path: /api/backoffice/orders/{orderId}

---

### Backoffice - Esecuzione Rimborso 
- Metodo: POST
- Path: /api/backoffice/orders/{orderId}/refund
- Richiesta idempotente per rimborso.

---

### Eventi asincroni via RabbitMQ 
- Payload JSON con order_id, event_type, timestamp, payload dettaglio evento.

## 6. Business logic 
- Workflow ordini: creazione ordine -> pagamento autorizzato -> decremento stock -> spedizione -> notifiche -> transizioni stato.
- Gestione rollback su pagamento fallito con ripristino stock.
- Gestione cancellazione ordine solo in stati permessi con compensazioni.
- Gestione rimborsi manuali via backoffice con integrazione Stripe e ripristino stock.
- Eventi di stato e notifiche inviati in modo asincrono e idempotente.
- Audit logging dettagliato per tutte le transizioni e operazioni critiche.
- Ottimizzazioni e retry automatici per errori temporanei.

## 7. Persistenza e integrazioni 
- PostgreSQL con schema normalizzato (orders, order_items, payments, shipments, stock, audit_logs).
- ORM JPA/Hibernate con repository pattern e gestione transazioni ACID.
- Integrazione Stripe per autorizzazione, cattura, rimborso con idempotenza e retry.
- Integrazione logistica esterna via REST e callback asincroni.
- Messaging RabbitMQ per eventi stato ordine con dead-letter queue e retry.
- Notifiche email tramite SendGrid e push notification esterne con opt-out.
- Crittografia dati sensibili e backup regolari per compliance GDPR.

## 8. Autenticazione e autorizzazione 
- Tutte le API HTTPS.
- Autenticazione JWT obbligatoria con token Bearer.
- Ruoli definiti: Cliente, Backoffice-operator.
- Validazione token e ruolo nell’API Gateway.
- Limitazioni di accesso per endpoint a seconda ruolo.
- Gestione sicurezza e validazione input/output per prevenzione injection.

## 9. Gestione errori 
- Validazione input con 400 Bad Request e messaggi chiari.
- Idempotenza con 409 Conflict su duplicati.
- Retry automatici con backoff esponenziale per errori temporanei.
- Circuit breaker per protezione da errori esterni persistenti.
- Logging strutturato e audit per errori permanenti e operazioni critiche.
- Risposte API uniformi con payload standard error:
```json
{
  "error_code": "string",
  "message": "string",
  "details": {}
}
```
- Dead-letter queue per messaggi falliti con alerting.

## 10. Strategia di test backend 

| Nome test                           | Tipo         | Cosa verifica                                      | Input                        | Output atteso                        |
|-----------------------------------|--------------|---------------------------------------------------|------------------------------|------------------------------------|
| testCreateOrderSuccess             | Unit         | Creazione ordine con dati validi                   | JSON ordine valido            | Risposta 201 con order_id          |
| testCreateOrderIdempotency         | Integration  | Idempotenza creazione ordine con stesso client_token | Doppia richiesta POST ordine  | 409 Conflict o stessa risposta    |
| testPaymentAuthorizationSuccess   | Integration  | Autorizzazione pagamento valida con Stripe         | Dati pagamento validi         | 200 OK con stato AUTHORIZED        |
| testPaymentAuthorizationFailure   | Integration  | Fallimento autorizzazione pagamento Stripe         | Dati errati o insufficiente fondi | 402 Payment Required             |
| testStockDecrementConcurrentAccess | Integration  | Concorrenza decremento stock con ordini paralleli  | Ordini concorrenti            | Nessun deadlock, stock coerente    |
| testRefundProcessBackoffice       | E2E          | Processo rimborso completo dall’API backoffice     | Richiesta rimborso corretta   | Stato ordine REFUNDED, stock ripristinato |
| testCancelOrderValidState         | Unit         | Cancellazione ordine valido                          | Richiesta cancel ordine       | 200 OK, ordine annullato           |
| testCancelOrderInvalidState       | Unit         | Tentativo cancellazione ordine non cancellabile    | Richiesta cancel in stato errato | Errore con messaggio appropriato |
| testRabbitMQEventProcessing       | Integration  | Elaborazione eventi asincroni stato ordine          | Messaggio evento valido       | Aggiornamento stato corretto       |
| testAuthJWTValidation             | Unit         | Validazione token JWT e controllo role              | Token validi e invalidi       | Accesso consentito o negato         |

## 11. Rischi tecnici 
- Gestione concorrenza nelle operazioni su stock con possibili race conditions.
- Dipendenza da integrazioni esterne (Stripe, logistica) soggette a failure persistenti.
- Complessità rollback e compensazioni incrociate pagamento-stock.
- Implementazione corretta e completa di idempotenza tramite token univoci.
- Gestione sicurezza accessi e compliance GDPR.
- Complessità nella gestione notifiche push e opt-out.
- Potenziali colli di bottiglia per performance non ottimizzate.
- Necessità di monitoraggio, tuning e alerting costante per mantenere SLA.

## 12. Struttura file proposta 

```
backend-order-management/
├── src/
│   ├── main/
│   │   ├── java/com/example/orders/
│   │   │   ├── api/
│   │   │   │   ├── OrderController.java        # Espone endpoint REST ordini
│   │   │   │   ├── PaymentController.java      # Gestione pagamenti e rimborso
│   │   │   │   ├── BackofficeController.java   # Endpoint backoffice gestione ordini e rimborsi
│   │   │   ├── service/
│   │   │   │   ├── OrderService.java            # Business logic ordini
│   │   │   │   ├── PaymentService.java          # Logica pagamenti Stripe
│   │   │   │   ├── RefundService.java           # Gestione rimborsi
│   │   │   │   ├── ShipmentService.java         # Integrazione logistica
│   │   │   │   ├── NotificationService.java     # Invio notifiche email/push
│   │   │   ├── repository/
│   │   │   │   ├── OrderRepository.java         # CRUD ordini
│   │   │   │   ├── PaymentRepository.java       # CRUD pagamenti
│   │   │   │   ├── StockRepository.java         # Gestione stock
│   │   │   │   ├── AuditLogRepository.java      # Persistenza audit logging
│   │   │   ├── messaging/
│   │   │   │   ├── EventPublisher.java          # Publisher eventi RabbitMQ
│   │   │   │   ├── EventConsumer.java           # Consumer e processing eventi
│   │   │   ├── config/
│   │   │   │   ├── SecurityConfig.java          # Configurazione sicurezza JWT, ruoli, CORS
│   │   │   │   ├── RabbitMQConfig.java          # Configurazione broker RabbitMQ
│   │   │   │   ├── StripeConfig.java             # Configurazione client Stripe
│   │   ├── resources/
│   │   │   ├── application.properties           # Configurazioni generali
│   │   │   ├── db/
│   │   │   │   ├── schema.sql                    # Script schema DB con tabelle e indici
│   │   │   ├── logback.xml                       # Configurazione logging strutturato
├── tests/
│   ├── unit/
│   │   ├── OrderServiceTest.java
│   │   ├── PaymentServiceTest.java
│   ├── integration/
│   │   ├── OrderCreationIntegrationTest.java
│   │   ├── PaymentAuthorizationIntegrationTest.java
│   ├── e2e/
│   │   ├── RefundProcessE2ETest.java
```

## 13. Piano di implementazione 
1. Setup progetto Java, configurazioni base (Jar, build, repository).
2. Definizione schema DB e script di creazione.
3. Implementazione moduli persistence e repository.
4. Sviluppo API REST con OrderController e PaymentController.
5. Implementazione Business Logic per creazione ordine, pagamento e stato.
6. Integrazione Stripe per autorizzazione, cattura, rimborso con gestione idempotenza.
7. Sviluppo messaging RabbitMQ publisher/consumer per eventi stato.
8. Implementazione Logging e Audit log.
9. Integrazione moduli spedizione e notifiche asincrone.
10. Creazione Backoffice API e gestione rimborsi.
11. Stesura test unitari, integration e E2E.
12. Setup CI/CD, monitoraggio, alerting.
13. Test load e tuning prestazioni.
14. Deployment e rilascio monitorato.

---

Proposta consolidata pronta per revisione QA e successiva implementazione.