MODULE: PM VERSION: 1
## 1. Obiettivo
Sviluppare un backend microservizio per la gestione ordini e-commerce, con integrazione a servizi esterni per pagamento, logistica e notifiche, garantendo scalabilità, resilienza, sicurezza e tracciabilità completa del processo di ordine.

## 2. Contesto e vincoli
- Microservizio autonomo e deployabile indipendentemente.
- Persistenza dati obbligatoria su PostgreSQL con transazioni ACID.
- Comunicazioni sicure via HTTPS con autenticazione e autorizzazione JWT.
- Integrazione con:
  - Gateway di pagamento Stripe per authorization, capture, rimborso.
  - Servizio logistico esterno per creazione spedizioni e tracking.
  - RabbitMQ per gestione asincrona aggiornamenti stato ordine.
  - SendGrid per invio notifiche email.
- Vincoli tecnologici: Java, Bear, REST API, PostgreSQL, JWT, SendGrid.
- Supporto picchi fino a 500 ordini al minuto.
- Latenza API < 200ms al 95° percentile.
- Idempotenza obbligatoria su tutte le operazioni critiche.
- Logging strutturato e audit logging per transizioni stato ordine.
- Resilienza con retry automatici su errori temporanei.
- Conformità GDPR per dati e backup.

## 3. Assunzioni
- Notifiche push gestite tramite servizi esterni standard con supporto minimo e possibilità di opt-out.
- In caso di pagamento fallito o annullato, rollback automatico delle risorse coinvolte (es. stock).
- Rimborso manuale effettuato esclusivamente da backoffice con conferma e integrazione Stripe.
- Conservazione audit log per minimo un anno.
- Autenticazione e controllo accessi API backoffice tramite JWT e ruoli.
- Non sono previsti flussi di sincronizzazione magazzino da sistemi esterni, focus solo su aggiornamenti da ordini nel microservizio.

## 4. Scope MVP
- Ricezione ordini tramite API REST con validazione lato server.
- Workflow ordine: validazione → pagamento (Stripe) → conferma → spedizione.
- Gestione asincrona aggiornamenti stato ordine tramite consumer RabbitMQ.
- Integrazione spedizioni e tracking con servizio logistica esterno.
- Invio notifiche email e push a ogni stato ordine (SendGrid + servizi push esterni).
- API backoffice per operatori: elenco ordini, dettaglio, gestione rimborso manuale.
- Gestione magazzino integrata con decremento stock a pagamento confermato e ripristino in caso di cancellazione.
- Audit log completo delle transizioni stato ordine.
- Meccanismi di retry e idempotenza per resilienza.
- Logging strutturato e monitoraggio performance.

## 5. Out of scope
- Sviluppo di frontend o interfacce utente.
- Gestione utenti o autenticazione client oltre backoffice.
- Supporto a metodi di pagamento diversi da Stripe.
- Gestione campagne marketing o promozioni.
- Automazione avanzata spedizioni oltre creazione/tracking.
- Supporto multi-magazzino o inventario manuale.
- Canali vendita offline o integrazione POS.
- Dettagli avanzati di sincronizzazione magazzino esterno.

## 6. Task tecnici ordinati
1. Progettazione schema dati in PostgreSQL con transazioni ACID.
2. Sviluppo API REST per ricezione ordini con validazione input.
3. Integrazione gateway Stripe per authorization/capture e rimborso.
4. Implementazione workflow ordine con stati e transizioni.
5. Implementazione consumer RabbitMQ per aggiornamenti asincroni.
6. Integrazione con servizio logistica per creazione spedizioni e tracking.
7. Implementazione invio notifiche email/push via SendGrid e servizi push.
8. Sviluppo API backoffice per visualizzazione ordini, dettaglio e gestione rimborso.
9. Gestione magazzino con aggiornamento stock basato su esito pagamento e cancellazioni.
10. Audit log di tutte le transizioni stato ordine con retention minima 1 anno.
11. Implementazione meccanismi idempotenza su operazioni critiche.
12. Gestione retry automatico su errori temporanei per payment e logistica.
13. Logging strutturato e monitoraggio di performance e anomalie.
14. Implementazione sicurezza: HTTPS, JWT con controllo ruoli.
15. Testing completo (happy path e scenari di errore).

## 7. Acceptance criteria
- API rispondono entro 200ms al 95° percentile.
- Capacità di gestire picchi fino a 500 ordini/minuto senza perdita dati.
- Tutte le operazioni critiche sono idempotenti, prevenendo duplicazioni.
- Workflow ordine completato con corretta integrazione pagamento e spedizione.
- Aggiornamenti stato ordine gestiti correttamente e asincroni tramite RabbitMQ.
- Notifiche email e push inviate a ogni cambio stato ordine.
- Backoffice consente visualizzazione ordini e gestione rimborso manuale integrato.
- Magazzino aggiornato correttamente a pagamento confermato e cancellazioni.
- Audit log completo e consultabile per almeno 1 anno.
- Retry automatico attivo e funzionale per errori temporanei.
- Comunicazioni e autenticazioni conformi a HTTPS/JWT/GDPR.
- Nessun crash o perdita di dati in caso di errori o timeout con servizi esterni.
- Test funzionali documentati e superati.

## 8. Rischi e punti aperti
- Dettagli implementativi non specificati su notifiche push (servizi, canali, opt-out).
- Gestione annunciata solo a livello di assunzione: rollback ordine su pagamento fallito da validare.
- Processo rimborso manuale da dettagliare con l'integrazione Stripe.
- Gestione cancellazione ordini in stati intermedi da definire con precisione per coerenza magazzino.
- Possibili problemi di concorrenza su aggiornamento stock in casi di ordini paralleli elevati.
- Gestione failure persistenti e fallback su Stripe o logistica da approfondire.
- Sicurezza e autorizzazioni backoffice da implementare conformemente a policy aziendali.