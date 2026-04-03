MODULE: BA VERSION: 1
## 1. Requisiti funzionali
- Ricezione ordini tramite API REST da frontend e canali esterni, con validazione degli input lato server.
- Integrazione con gateway di pagamento Stripe per processi di authorization e capture del pagamento.
- Workflow ordine completo: validazione dati ordine → pagamento → conferma ordine → gestione spedizione.
- Gestione asincrona degli eventi di aggiornamento stato ordine tramite consumer su RabbitMQ.
- Integrazione con servizio esterno di logistica per creazione spedizioni e tracciamento (tracking).
- Invio notifiche email e push al cliente ad ogni cambio di stato ordine, tramite integrazione con SendGrid.
- API di backoffice per operatori: visualizzazione lista ordini, dettaglio ordine, gestione rimborso manuale.
- Gestione magazzino integrata: decremento stock su conferma pagamento, ripristino stock in caso di cancellazione.
- Audit log di tutte le transizioni di stato ordine per tracciabilità e compliance.
- Funzionalità di retry automatico in caso di errori temporanei nei servizi payment e logistica.
- Idempotenza garantita per tutte le operazioni critiche per evitare duplicazioni e incongruenze.

## 2. Requisiti non funzionali
- Latenza delle API inferiore a 200 ms al 95° percentile per garantire buona esperienza utente.
- Supporto a picchi di carico fino a 500 ordini al minuto, con adeguata scalabilità orizzontale.
- Persistenza dati con PostgreSQL usando transazioni ACID per garantire coerenza e integrità dati.
- Comunicazioni sicure via HTTPS; utilizzo di JWT per autenticazione e autorizzazione degli endpoint.
- Logging strutturato e tracciabilità end-to-end completa per ogni ordine.
- Resilienza ai guasti con meccanismi di retry e idempotenza.
- Backup e protezione dei dati in ottica GDPR e sicurezza.
- Monitoraggio performance e alerting per anomalie operative.

## 3. Ambiguità e domande aperte
- Dettaglio sulla gestione esatta delle notifiche push (tecnologia, canali supportati, frequenza, opt-out) non specificato. Assunzione: supporto minimo a notifiche push via servizi esterni standard.
- Comportamento esatto in caso di pagamento fallito o annullato non dettagliato (ordine mantiene stato o rollback completo?). Assunzione: rollback automatico delle risorse se pagamento non confermato.
- Dettagli sul processo di rimborso manuale (range autorizzato, integrazione con payment gateway) non esplicitati. Assunzione: rimborso gestito solo tramite backoffice con conferma manuale e integrazione tramite Stripe API.
- Non definito il livello di storicizzazione e retention degli audit log. Assunzione: conservazione audit log minimo 1 anno.
- Sicurezza e accessi alle API di backoffice non dettagliati, presumibile uso JWT con ruoli.
- Non specificate le modalità di sincronizzazione o aggiornamento del magazzino da altri sistemi esterni.

## 4. Edge case e scenari limite
- Gestione di ordini duplicati inviati ripetutamente tramite retry API: confermata idempotenza come mitigazione.
- Eventuali timeout o indisponibilità prolungata di Stripe o servizio logistica: presenza di meccanismi retry ma da dettagliare per casi di failure persistente.
- Cancellazione ordini in stato intermedio (pagamento in corso o spedizione in corso): scenario da definire per coerenza stock e flusso.
- Gap temporanei o perdita messaggi nella queue RabbitMQ e loro effetti sullo stato ordine.
- Gestione ordini di grande volume in parallelo che impattano sul magazzino (concorrenza e lock risorse).
- Eventuali problemi di scalabilità in caso di superamento picco ordini 500 ordini/minuto.

## 5. Dipendenze e vincoli
- Dipendenza dal gateway Stripe per processo pagamento e rimborso.
- Dipendenza dal servizio logistica esterno per spedizione e tracking.
- Dipendenza da RabbitMQ per gestione asincrona eventi di aggiornamento stato.
- Vincolo di utilizzo PostgreSQL con transazioni ACID per garantire coerenza.
- Vincolo architetturale microservizio autonomo e deployabile indipendentemente.
- Obbligo di idempotenza su operazioni critiche strettamente collegata a retry e resilienza.
- Vincolo stack tecnologico: Java, Bear, REST API, PostgreSQL, JWT, SendGrid.
- Sicurezza garantita tramite HTTPS e autenticazione JWT, necessario adeguato controllo accessi.

## 6. Requisiti esclusi esplicitamente
- Non è richiesto sviluppo di frontend o interfacce utente, solo backend REST API.
- Non è richiesta gestione utenti o autenticazione client oltre al backoffice.
- Non è richiesto supporto a metodi di pagamento diversi da Stripe.
- Non è richiesta gestione di campagne marketing o promozioni.
- Non è richiesta automazione spedizioni oltre alla creazione e tracking tramite servizio esterno.
- Non è richiesto supporto a più magazzini o scenari multi-warehouse.
- Non è prevista gestione avanzata di stock non legata a ordini (es. inventario manuale).
- Non sono previsti canali di vendita offline o integrazione POS.

Il documento chiarisce e struttura in modo completo i requisiti, includendo impliciti importanti come sicurezza, validazione, idempotenza e resilienza.