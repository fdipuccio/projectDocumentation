```
MODULE: BA VERSION: 1

## 1. Requisiti funzionali

- **Ricezione ordini tramite API REST**
  - Creare endpoint RESTful per la ricezione degli ordini.
  - Assicurarsi che gli ordini contengano tutte le informazioni necessarie per la corretta elaborazione.

- **Integrazione con gateway di pagamento (Stripe)**
  - Implementare flussi di authorization e capture per gestire i pagamenti con Stripe.
  - Gestire le transazioni fallite e garantire l'idempotenza nelle operazioni di pagamento.

- **Workflow di elaborazione ordine**
  - Sequenza di operazioni: validazione → pagamento → conferma → spedizione.
  - Gestione di eventuali errori con meccanismi di compensazione.

- **Consumer asincrono per eventi di aggiornamento stato**
  - Utilizzare RabbitMQ per la gestione di eventi asincroni di aggiornamento stato ordine.

- **Integrazione con servizio esterno di logistica**
  - Creare e ottenere tracking delle spedizioni dal servizio di logistica esterno.

- **Notifiche email/push al cliente**
  - Inviare notifiche su ogni cambio di stato ordine tramite SendGrid.
  - Supportare notifiche sia email sia push.

- **API di backoffice per operatori**
  - Fornire funzionalità per elenco ordini, dettagli ordine e gestione rimborsi.
  - Implementare endpoint per funzioni amministrative.

- **Gestione magazzino**
  - Scalare stock al pagamento confermato.
  - Ripristinare stock in caso di cancellazione dell'ordine.

## 2. Requisiti non funzionali

- **Sicurezza**: Autenticazione e autorizzazione tramite JWT per proteggere le API.
- **Performance**: Il sistema deve rispondere in modo efficiente alle richieste di API e avere tempi di latenza minimi.
- **Scalabilità**: Supporto per aumentare la capacità di gestire richieste crescenti tramite l'architettura a microservizi.
- **Logging**: Implementare audit log dettagliati per tutte le transizioni di stato ordine.
- **Monitoraggio**: Predisporre strumenti per monitorare l'uptime, le performance del sistema e le code di messaggi.

## 3. Ambiguità e domande aperte

- **Notifiche push**: È necessario definire il provider da usare per le notifiche push (e.g., Firebase, altro)?
- **Authorization details with Stripe**: Quali parametri specifici dovremmo gestire per l'integrazione con Stripe?
  
Assunzione: Utilizzeremo le opzioni di default e best practice per le aree non specificate.

## 4. Edge case e scenari limite

- Cosa succede se un ordine viene ricevuto senza alcune informazioni essenziali?
- Come gestire ordini duplicati o pagamento doppio nei casi di retry?
- Gestione della restituzione di fondi nel caso in cui l'integrazione logistica fallisca.

## 5. Dipendenze e vincoli

- La ricezione degli ordini tramite API è dipendente dall'avere endpoint sicuri e validati.
- Gli ordini non possono proseguire in stato "spedizione" senza conferma dal sistema logistico.
- La gestione delle notifiche richiede un'integrazione stabile con SendGrid e definizione chiara dei trigger di stato.
- Vincoli di transazioni ACID su PostgreSQL per operazioni critiche, richiedendo un'attenzione aggiuntiva per le garanzie di rollback.

## 6. Requisiti esclusi esplicitamente

- Non sono richieste integrazioni con altri metodi di pagamento oltre a Stripe.
- Non è menzionato un front-end per i clienti finali, quindi non è incluso in questo progetto.
- Non è richiesta la gestione delle promozioni o scontistiche sugli ordini.

```