MODULE: BACKEND VERSION: 1

## 1. Obiettivo backend

Progettare e realizzare un sistema backend in grado di gestire il flusso operativo di ordini e-commerce con integrazioni a servizi esterni e garantendo sicurezza e scalabilità, utilizzando uno stack tecnologico basato su Java, RabbitMQ, PostgreSQL, e Bear framework.

## 2. Assunzioni tecniche

- Comunicazione attraverso eventi asincroni per coordinare i microservizi tramite RabbitMQ.
- Persistenza su PostgreSQL con gestione transazionale (ACID).
- Sicurezza per la comunicazione mediante protocolli sicuri (HTTPS, JWT).
- Utilizzo di Bear come ORM per un accesso strutturato al database.
- Integrazione di pagamenti gestita da Stripe tramite chiavi API.
- Notifiche email attraverso SendGrid senza memorizzare contenuti sensibili di posta nel sistema.

## 3. Architettura backend

L'architettura si basa su microservizi con comunicazione asincrona. I componenti principali includono:
- **Order Service**: Gestione e orchestrazione del ciclo di vita dell'ordine.
- **Payment Service**: Interfaccia verso Stripe per le transazioni.
- **Notification Service**: Gestisce la distribuzione delle notifiche email tramite SendGrid.
- **Logistics Service**: Comunicazione con API di logistici esterni per il tracking e aggiornamento di stato delle spedizioni.

## 4. Moduli e responsabilità

- **Order Module**: Gestione CRUD degli ordini, tracking dello stato dell'ordine.
- **Payment Module**: Integrazione e transazione con Stripe, retries e gestione errori.
- **Notification Module**: Configurazione e invio delle email tramite SendGrid.
- **Logistics Module**: Gestisce l'interazione con i servizi di logistica e aggiorna lo stato della spedizione.

## 5. Worker / Job Design

### OrderProcessorWorker
- **Trigger**: Ricezione nuovo evento ordine da RabbitMQ.
- **Message/Event Schema**: JSON con dettagli ordine (ID, stato, timestamp).
- **Flusso di Elaborazione**: Validazione -> Pagamento -> Conferma -> Invio notifica -> Integrazione logistica.

### PaymentRetryWorker
- **Trigger**: Evento di fallimento transazione.
- **Message/Event Schema**: JSON con dettagli errore transazione.
- **Flusso di Elaborazione**: Gestione retry con backoff esponenziale -> In caso di ripetuto fallimento invio a DLQ.

## 6. Business logic

- Ogni ordine attraversa uno stato di verifica e pagamento prima di essere confermato.
- Centralizzazione del controllo di transazione per evitare perdite dati su errori.
- Ridondanza e idempotency assicurate per l'integrazione con servizi esterni (Stripe, Logistica).

## 7. Persistenza e integrazioni

- Database PostgreSQL con connessione gestita da HikariCP per la ottimizzazione delle prestazioni.
- Integrazione con Stripe tramite endpoint sicuri.
- Notifiche di stato dipendenti da RabbitMQ e SendGrid.

## 8. Idempotency e Error Handling

- **Strategia Retry**: Tre tentativi su errori con logica di backoff esponenziale.
- **DLQ**: Implementazione di una coda separata per il fallimento transazioni irrecuperabili.
- **Deduplicazione**: Attraverso token univoci rilevati nei payload.
- **Alerting**: Set up di alert basati su eventi di fallimento critico.

## 9. Autenticazione e autorizzazione

JWT utilizzati per gestire sessioni sicure su tutti i processi microservizi critici.

## 10. Strategia di test backend

### Order Module Test
- **Nome**: OrderProcessingTest
- **Tipo**: Integration
- **Cosa verifica**: Intero flusso di elaborazione ordine
- **Input**: Simulazione ordine valido
- **Output**: Stato ordine finale e registrazione in DB

### Payment Module Test
- **Nome**: PaymentTransactionTest
- **Tipo**: Unit
- **Cosa verifica**: Transazione con Stripe
- **Input**: Dettagli pagamento simulato
- **Output**: Risposta positiva da Stripe

## 11. Rischi tecnici

- Rischi di rate limit da Stripe richiedono gestione rigida della frequenza delle richieste.
- Dipendenze strette da servizi esterni possono creare colli di bottiglia.

## 12. Struttura file proposta

```
/app
  /order
    order_service.py       # Logica di gestione ordini
  /payment
    payment_integration.py # Integrazione con Stripe
  /notifications
    notification_dispatch.py # Integrazione SendGrid
  /logistics
    logistics_connector.py # Interface API esterne
/tests
  test_order_module.py     # Test del modulo ordini
  test_payment_module.py   # Test del modulo pagamenti
```

## 13. Piano di implementazione

1. Configurazione ambiente e setup DB
2. Implementare workflow ordini
3. Integrazione con Stripe e gestione errori
4. Implementare flusso email SendGrid
5. Strategia di retry e DLQ
6. Testing completo e release.