MODULE: PM VERSION: 1
## 1. Obiettivo
Sviluppare un microservizio backend per la gestione degli ordini e-commerce che consenta la ricezione ordini tramite API REST sicure, integri il gateway di pagamento Stripe per authorization e capture, gestisca un workflow end-to-end degli ordini (validazione, pagamento, conferma, spedizione), consumi eventi asincroni da RabbitMQ per aggiornamenti stato ordine, integri un servizio esterno di logistica per spedizioni e tracking, invii notifiche email/push tramite SendGrid, esponga API di backoffice per operatori e gestisca il magazzino scalando e ripristinando stock, garantendo resilienza, sicurezza, scalabilità e completa tracciabilità.

## 2. Contesto e vincoli
- Architettura microservizi deployabile in modo indipendente.
- Persistenza dati su PostgreSQL con transazioni ACID.
- Comunicazione asincrona tramite RabbitMQ.
- Sicurezza delle API REST tramite JWT Bearer Token.
- Idempotenza obbligatoria per tutte le operazioni critiche (creazione ordine, chiamate a Stripe, logistica).
- Retry automatico con backoff per errori transitori su integrazioni (Stripe, logistica, SendGrid).
- Logging strutturato e audit log immutabile minimo per un anno.
- Stack tecnologico: Java, REST API, Bear, PostgreSQL, JWT.
- Conforme a normative GDPR e PCI-DSS per la gestione dati sensibili.
- Non è richiesto frontend o gestione utenti e ruoli oltre JWT.
- Stripe è l’unico gateway di pagamento supportato.
- Notifiche solo via email e push tramite SendGrid.
- Gestione backoffice limitata a API per operatori autenticati.

## 3. Assunzioni
- Eventi da RabbitMQ relativi ad aggiornamenti stato ordine con formato JSON standard.
- Notifiche push previste come estensioni future; MVP implementa email tramite SendGrid.
- Operatori backoffice gestiti da sistema esterno di identity management, con ruolo base di amministratore per rimborsi.
- La cancellazione ordine è manuale da operatori o automatica predefinita, con ripristino stock.
- Durata autorizzazione pagamento Stripe considerata temporanea e coerente con workflow interno.
- Strategie di retry prevedono numero limitato di tentativi con intervalli crescenti (backoff).
- Compliance normativa GDPR e PCI-DSS applicata nelle implementazioni di sicurezza e dati.

## 4. Scope MVP
- API REST per ricezione e validazione ordini con idempotenza.
- Integrazione con Stripe per authorization e capture pagamento con retry e idempotenza.
- Workflow ordine con stati Validazione → Pagamento → Conferma → Spedizione e audit log.
- Consumer asincrono RabbitMQ per eventi stato ordine con gestione duplicati e ordine eventi.
- Integrazione base con servizio logistico per creazione spedizioni e aggiornamento tracking, con retry.
- Invio notifiche email al cliente a ogni cambio stato tramite SendGrid con retry.
- API backoffice per operatori: lista ordini, dettaglio e rimborso manuale con autorizzazione JWT.
- Gestione magazzino: scalare stock a pagamento confermato, ripristino su cancellazione o errore.
- Logging strutturato e immutabile per tracciare tutte le transizioni di stato.

## 5. Out of scope
- Sviluppo frontend cliente o backoffice.
- Integrazione con gateway pagamento diversi da Stripe.
- Sistema di gestione utenti e ruoli oltre JWT per backoffice.
- Sistema complesso di inventario diverso da scalare/ripristino stock.
- Reportistica avanzata o analisi ordini.
- Gestione multi-valuta o multi-paese.
- Canali di notifica diversi da email/push tramite SendGrid.

## 6. Task tecnici ordinati
1. Progettazione e sviluppo API REST per ricezione ordini con validazione input e idempotenza.
2. Implementazione integrazione Stripe per autorizzazione e capture pagamento con gestione retry e idempotenza.
3. Definizione e coding workflow stato ordine con audit log immutabile.
4. Sviluppo consumer asincrono RabbitMQ per consumare e processare eventi stato ordine con de-duplicazione e ordering.
5. Integrazione con servizio esterno logistica per creazione spedizioni, tracking e aggiornamento stato ordine con retry.
6. Implementazione invio notifiche email via SendGrid con gestione retry e logging degli errori.
7. Sviluppo API di backoffice per operatori con autenticazione JWT: lista ordini, dettaglio, rimborso manuale e autorizzazione.
8. Implementazione gestione magazzino: scalatura stock a pagamento confermato, ripristino stock su cancellazione o errore.
9. Sviluppo modulo di logging strutturato e audit log immutabile con retention minima un anno.
10. Configurazione sicurezza API tramite JWT Bearer Token e sanificazione input.
11. Testing end-to-end incluse resilience su retry e idempotenza.
12. Documentazione tecnica API, workflow e operazioni di backoffice.

## 7. Acceptance criteria
- API REST ricezione ordine risponde entro 200ms al 95° percentile sotto carico fino a 500 ordini/minuto.
- Validazione input ordine completa con error handling chiaro e idempotenza su creazione ordine.
- Chiamate a Stripe per pagamento autorizzazione e cattura effettuate con retry e idempotenza senza duplicati.
- Workflow ordine dimostra transizioni corrette e audit log immutabile tracciate tutte le operazioni significative.
- Consumer RabbitMQ processa eventi ordine in modo idempotente e gestisce eventi duplicati e fuori sequenza.
- Integrazione logistica permette creazione spedizioni e aggiornamento tracking con retry su errori transitori.
- Notifiche email inviate correttamente tramite SendGrid su ogni cambio stato con retry su errori.
- API backoffice protette da JWT consentono visualizzazione lista ordini, dettagli e gestione rimborsi manuali con autorizzazione.
- Magazzino scala e ripristina stock coerentemente con stato pagamenti e cancellazioni.
- Logging strutturato e audit log conservati per almeno un anno e immutabili.
- Rispetto di vincoli sicurezza JWT, GDPR e PCI-DSS per dati sensibili.
- Documentazione completa per API, backoffice e integrazioni.

## 8. Rischi e punti aperti
- Definizione dettagliata degli eventi RabbitMQ: struttura esatta e tipologie da confermare.
- Canali, formati e protocolli notifiche push da definire con eventuale piano estensione futuro.
- Gestione dettagliata permessi operatori backoffice e sistema di identity management da confermare.
- Parametri fine tuning per retry (numero tentativi, intervalli backoff) da validare.
- Gestione scenario cancellazione ordine: automatica/manuale e impatti su stock e pagamenti da chiarire.
- Procedures di cleanup per pagamenti autorizzati non catturati interventi e timeout.
- Concorrenza e potenziali race condition sulle transizioni stato ordine e gestioni stock da monitorare.
- Compliance GDPR e PCI-DSS da integrare in dettaglio nelle implementazioni e test.