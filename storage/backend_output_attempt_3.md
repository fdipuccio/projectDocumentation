MODULE: BACKEND VERSION: 1

## 1. Obiettivo backend
Il microservizio backend è progettato per gestire l'intero ciclo di vita degli ordini e-commerce, includendo creazione, validazione, pagamento, gestione stock, spedizione, e rimborso. Deve garantire scalabilità, resilienza, sicurezza, tracciabilità completa tramite audit log immutabili, integrazione con servizi esterni (Stripe, logistica, SendGrid, RabbitMQ), e rispondere a requisiti di compliance PCI-DSS e GDPR.

## 2. Assunzioni tecniche
- Implementazione in Java su architettura a microservizi event-driven.
- Persistenza su PostgreSQL con supporto ACID e transazioni esplicite.
- Comunicazione tramite API REST HTTPS con payload JSON.
- Autenticazione JWT Bearer Token per tutte le API ad eccezione dell'endpoint creazione ordine in MVP.
- Idempotenza obbligatoria tramite header `X-Idempotency-Key` per operazioni critiche.
- Retry automatico con backoff esponenziale su errori transitori per integrazioni esterne.
- Log strutturato e audit log immutabile per tutte le transizioni di stato degli ordini.
- Controlli concorrenziali tramite lock pessimista/ottimistico per evitare race condition.
- Compliance GDPR e PCI-DSS per gestione dati sensibili e sicurezza.
- Obiettivo di performance: <200ms di latenza al 95° percentile sotto carico fino a 500 ordini al minuto.

## 3. Architettura backend
Architettura microservizi con pattern event-driven e layered design composta da:
- API REST Controller per gestione richieste client.
- Moduli di dominio per business logic dell’ordine, pagamento, stock, logistica.
- Repository per accesso e manipolazione dati persistenti su PostgreSQL.
- Client esterni per Stripe, servizio logistica e SendGrid con meccanismi di retry/idempotenza.
- Consumer RabbitMQ per gestione eventi asincroni con controllo idempotenza e ordering garantito.
- Middleware di sicurezza per autenticazione e autorizzazione JWT, validazione input, rate-limiting.
- Audit Logger per registrazione immutabile di tutte le transizioni e operazioni critiche.

## 4. Moduli e responsabilità
- **Ordine Management Service:** gestione flusso ordini, validazione, stato ordine, rimborso.
- **Payment Integration Module:** comunicazioni con Stripe per autorizzazione e capture.
- **Warehouse Stock Repository:** gestione concorrenziale dello stock con lock e rollback.
- **Logistics Integration Module:** interazione con servizio esterno spedizioni e tracking.
- **Notification Service:** invio email e notifiche tramite SendGrid.
- **RabbitMQ Consumer:** elaborazione eventi asincroni, gestione aggiornamenti stato ordine.
- **Audit Logger:** persistenza immutabile degli eventi di stato e operazioni.
- **Security Middleware:** controllo JWT, autorizzazioni ruoli, sanitizzazione delle richieste.

## 5. API principali

### 5.1 POST `/api/v1/orders`
- **Descrizione:** Creazione nuovo ordine e-commerce.
- **Autenticazione:** NO (MVP)  
- **Headers:** 
  - `Content-Type: application/json`
  - `X-Idempotency-Key: string` (UUID/hash unico)
- **Request Body:**
```json
{
  "customer": {
    "email": "cliente@example.com",
    "name": "Mario Rossi"
  },
  "items": [
    {"sku": "ABC123", "quantity": 2, "unit_price": 15.50},
    {"sku": "XYZ789", "quantity": 1, "unit_price": 45.00}
  ],
  "payment": {
    "method": "stripe",
    "currency": "EUR",
    "total_amount": 76.00
  },
  "shipping_address": {
    "street": "Via Roma 1",
    "city": "Milano",
    "postal_code": "20100",
    "country": "IT"
  }
}
```
- **Response 201 Created:**
```json
{
  "order_id": "uuid-ordine",
  "status": "Validation",
  "created_at": "2024-06-01T12:00:00Z"
}
```
- **Errori 400/409 (idempotenza):**
```json
{
  "error_code": "ORDER_VALIDATION_FAILED",
  "message": "Messaggio di dettaglio errore"
}
```

### 5.2 GET `/api/v1/orders/{orderId}`
- **Descrizione:** Recupero dettaglio ordine.
- **Autenticazione:** Sì (JWT Bearer Token).
- **Response 200 OK:**
```json
{
  "order_id": "uuid-ordine",
  "status": "Confirmed",
  "customer": { "email": "...", "name": "..." },
  "items": [...],
  "payment": { "status": "Captured", "amount": 76.00, "currency": "EUR" },
  "shipping": { "carrier": "DHL", "tracking_code": "TRACK123", "status": "Shipped" },
  "timestamps": {
    "created": "...",
    "validated": "...",
    "paid": "...",
    "confirmed": "...",
    "shipped": "..."
  }
}
```

### 5.3 GET `/api/v1/orders`
- **Descrizione:** Lista ordini filtrabile per stato e range date.
- **Autenticazione:** Sì (JWT Bearer Token).
- **Query params:** status, fromDate, toDate
- **Response 200 OK:**
```json
[
  {
    "order_id": "uuid",
    "status": "Paid",
    "created_at": "...",
    "total_amount": 76.00,
    "customer_email": "example@example.com"
  },
  ...
]
```

### 5.4 POST `/api/v1/orders/{orderId}/refund`
- **Descrizione:** Richiesta rimborso manuale ordine.
- **Autenticazione:** Sì (JWT Bearer Token, ruolo admin).
- **Headers:** `X-Idempotency-Key`
- **Request Body:**
```json
{
  "reason": "Cliente ha richiesto rimborso parziale"
}
```
- **Response 200 OK:**
```json
{
  "refund_id": "uuid-rimborso",
  "status": "RefundRequested",
  "order_id": "uuid-ordine",
  "processed_at": "2024-06-01T14:00:00Z"
}
```

### 5.5 GET `/api/v1/warehouse/stock/{sku}`
- **Descrizione:** Consultazione stock magazzino per SKU.
- **Autenticazione:** Sì (JWT Bearer Token).
- **Response 200 OK:**
```json
{
  "sku": "ABC123",
  "available_stock": 125,
  "reserved_stock": 20
}
```

## 6. Business logic
- Creazione ordine con validazione input, controllo idempotenza.
- Workflow stato ordine: Validazione → Pagamento (authorization + capture Stripe) → Conferma → Spedizione.
- Scalatura stock solo a conferma pagamento con gestione concorrenza via lock.
- Gestione retry automatici e idempotenza per tutte le operazioni esterne (Stripe, logistica, notifiche).
- Logging immutabile di tutte le transizioni di stato e azioni critiche.
- Workflow asincrono gestito con RabbitMQ per aggiornamenti stato e notifiche.
- Gestione rimborso tramite chiamate coordinate a Stripe e aggiornamento stato ordine/stock.

## 7. Persistenza e integrazioni
- Backend con PostgreSQL per dati persistenti (ordini, stock, audit log).
- Tabelle ottimizzate con indici su ID ordine, stato, timestamp e idempotency key.
- Transazioni ACID esplicite per garantire consistenza operazioni critiche.
- Integrazione Stripe per payments (authorization, capture, refunds).
- Integrazione con servizio logistica per spedizioni, tracking.
- Integrazione SendGrid per notifiche email.
- Eventi asincroni processati tramite consumer RabbitMQ con gestione di idempotenza e ordering.

## 8. Autenticazione e autorizzazione
- JWT Bearer Token obbligatorio su tutte le API ad eccezione endpoint creazione ordine (MVP).
- Token contiene claim (userId, ruolo).
- Middleware effettua validazione token, autorizzazione ruoli e sanitizzazione input.
- Operazioni sensibili (rimborso) limitate a ruoli amministrativi.
- Nessuna gestione utenti interna, identity management affidato a sistemi esterni.

## 9. Gestione errori
- Errori validazione input con HTTP 400 e messaggi JSON dettagliati.
- Errori duplicazione via idempotency restituiscono 409 con stato attuale.
- Errori integrazione esterna gestiti con retry e backoff.
- Errori server generici con 500 e messaggio generico per sicurezza.
- Logging dettagliato di ogni errore per diagnosi e audit.
- Rollback transazioni su errori critici per mantenere consistenza dati.
- Alert operativi per errori critici.

## 10. Strategia di test backend

| Nome Test                   | Tipo        | Verifica                                                | Input                          | Output Atteso                                                |
|----------------------------|-------------|---------------------------------------------------------|--------------------------------|-------------------------------------------------------------|
| CreateOrder_ValidRequest    | Unit        | Validazione input e successo creazione ordine           | Richiesta POST `/orders` valida | 201 Created con order_id e stato `Validation`               |
| CreateOrder_DuplicateIdemp  | Integration | Controllo idempotency key su creazione ordine doppia    | Stessa request con stessa key   | 409 Conflict con stato ordine corrente                      |
| GetOrder_ValidId            | Unit        | Recupero dettagli ordine con autorizzazione valida      | GET `/orders/{orderId}` con JWT | 200 OK con dati ordine                                       |
| RefundOrder_AuthorizedUser  | Integration | Rimborso manuale con idempotency e controllo ruoli      | POST `/orders/{orderId}/refund` con token admin | 200 OK con dettaglio rimborso                                 |
| StockManagement_Concurrency | Integration | Gestione concorrenza su aggiornamento stock              | Multiple decrement/increment   | Nessuna race condition, stock coerente                       |
| PaymentIntegration_Retry    | Unit        | Retry e idempotenza chiamate Stripe                      | Simulazione errori transitori  | Chiamata ripetuta con successo o errore gestito             |
| RabbitMQConsumer_OrderEvents| Integration | Elaborazione eventi ordine con idempotenza e ordering   | Eventi duplicati e out of order| Eventi processati una sola volta nell'ordine corretto       |

## 11. Rischi tecnici
- Schema evento RabbitMQ incompleto o instabile può causare inconsistenze.
- Race condition e conflitti concorrenza su stock e stato ordine richiedono attenzione a lock e transazioni.
- Parametri retry da tarare per bilanciare resilienza e latenza.
- Compliance GDPR e PCI-DSS richiede test approfonditi e auditing sicuro.
- Gestione timeout e cleanup pagamenti non catturati deve essere definita.
- Gestione efficiente del logging immutabile per storage e performance.
- Progettazione estensibile per future notifiche push senza impatti MVP.
- Sicurezza token JWT e accessi backoffice controllati rigidamente.

## 12. Struttura file proposta

```
/src
 ├── /api
 │    ├── OrdersController.java          # Gestione API REST ordini
 │    ├── WarehouseController.java       # API magazzino/stock
 │    ├── SecurityMiddleware.java        # Middleware JWT e sicurezza
 ├── /service
 │    ├── OrderManagementService.java    # Logica business ordini
 │    ├── PaymentIntegrationService.java # Comunicazione Stripe
 │    ├── LogisticsIntegrationService.java # Interazione spedizioni
 │    ├── NotificationService.java       # Invio email con SendGrid
 │    ├── RabbitMQConsumerService.java   # Gestione eventi asincroni
 ├── /repository
 │    ├── OrderRepository.java           # Accesso DB ordini, idempotenza
 │    ├── StockRepository.java           # Gestione stock con lock
 │    ├── AuditLogRepository.java        # Insert append-only log audit
 ├── /model
 │    ├── Order.java                     # Entità ordine
 │    ├── Stock.java                     # Entità stock magazzino
 │    ├── AuditLog.java                  # Entità log immutabili
 │    ├── Payment.java                   # Entità pagamento
 ├── /integration
 │    ├── StripeClient.java              # Client Stripe con retry/idempotenza
 │    ├── LogisticsClient.java           # Client servizio spedizioni
 │    ├── SendGridClient.java            # Client invio email
 ├── /config
 │    ├── AppConfig.java                 # Configurazioni applicative
 ├── /util
 │    ├── JsonValidator.java             # Validazione JSON input
 │    ├── IdempotencyHelper.java         # Gestione idempotency key
 └── /exception
      ├── ApiException.java              # Gestione errori API custom
      ├── RetryException.java            # Eccezioni retry
```

## 13. Piano di implementazione
1. Setup ambiente sviluppo e definizione schemi DB PostgreSQL.
2. Implementazione Repository e modello entità con transazioni ACID.
3. Sviluppo moduli core OrderManagementService e StockRepository con lock.
4. Implementazione API REST Controller con validazioni e middleware sicurezza.
5. Integrazione Stripe client con retry e idempotenza.
6. Implementazione workflow creazione ordine, pagamento e aggiornamenti stato.
7. Integrazione servizi di logistica e notifiche.
8. Sviluppo RabbitMQ consumer per gestione eventi asincroni.
9. Implementazione audit log immutabile.
10. Scrittura test unitari e di integrazione per API, business logic e integrazioni.
11. Deployment ambiente test e stress test performance.
12. Fix bug, ottimizzazione, documentazione finale e preparazione per QA.

---

Questo documento consolidato riunisce i contributi degli specialist backend ed è pronto per la fase di QA e sviluppo.