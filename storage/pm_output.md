MODULE: PM VERSION: 1

## 1. Obiettivo
Sviluppare un sistema backend per la gestione degli ordini di e-commerce con integrazione a servizi esterni, con funzionalità che coprono ricezione degli ordini, gestione dei pagamenti, aggiornamento dello stato degli ordini, gestione logistica, notifiche agli utenti e funzioni di backoffice per operatori.

## 2. Contesto e vincoli
- Vincoli tecnici: L'intero sistema deve essere sviluppato utilizzando Java, PostgreSQL, RabbitMQ e JWT per autenticazione e autorizzazione. Implementazione Microservice Cropieza packaging. Vincolo di utilizzare esclusivamente Stripe per i pagamenti e SendGrid per le notifiche.
- Vincoli di business: Integrazione obbligatoria con Stripe e SendGrid.
- Comunicazione asincrona con alti microservizi tramite RabbitMQ.

## 3. Assunzioni
- Il sistema di pagamento utilizzerà chiavi API standard di Stripe per l'autenticazione.
- Si ipotizza l'uso di API REST standard per la comunicazione con i servizi di logistica.
- Le notifiche al cliente avverranno almeno tramite email e push.
- Non è richiesta un'interfaccia utente per clienti o operatori all'interno del sistema backend.

## 4. Scope MVP
- Implementazione di un'API REST per la ricezione degli ordini.
- Integrazione di pagamento con Stripe per l'autorizzazione e la cattura dei pagamenti.
- Workflow di elaborazione ordini: validazione, pagamento, conferma e spedizione.
- Consumer asincrono RabbitMQ per la gestione dell'aggiornamento dello stato degli ordini.
- Integrazione con un servizio esterno di logistica per la creazione delle spedizioni e l'aggiornamento del tracking.
- Notifiche al cliente con integrazione SendGrid.
- API di backoffice per visualizzazione e gestione ordini inclusi i rimborsi manuali.
- Gestione dello stock di magazzino in base agli ordini pagati e annullati.

## 5. Out of scope
- Supporto multivaluta non richiesto nel backend.
- Non è richiesta la gestione di sconti o promozioni all'interno del workflow degli ordini.
- Non è previsto lo sviluppo di interfacce utente.

## 6. Task tecnici ordinati
1. Definire l'architettura del microservizio e configurare l'ambiente di sviluppo.
2. Sviluppare l'API REST per la ricezione e gestione degli ordini, inclusa la validazione dell'input.
3. Integrare l'API con il gateway di pagamento Stripe.
4. Implementare il workflow di elaborazione degli ordini.
5. Configurare un consumer RabbitMQ per l'aggiornamento asincrono degli ordini.
6. Integrare con il servizio logistico esterno via API REST.
7. Configurare SendGrid per notifiche email/push ai clienti.
8. Sviluppare le API per supportare il backoffice con funzioni di gestione ordini e rimborsi.
9. Implementare i meccanismi di aggiornamento dello stock di magazzino.
10. Assicurare la sicurezza dei dati tramite JWT e crittografia dove necessario.

## 7. Acceptance criteria
- L'API REST è in grado di ricevere e processare ordini correttamente e offre una risposta in tempo reale.
- L'integrazione con Stripe deve gestire i pagamenti in modo sicuro e tracciabile.
- Il sistema è in grado di elaborare workflow di ordine completi e gestire rollback nei casi di errore.
- Il consumer RabbitMQ aggiorna lo stato degli ordini correttamente in modo asincrono.
- L'integrazione logistica rispetta i termici di creazione spedizione e aggiornamento tracking.
- Le notifiche ai clienti vengono inviate correttamente via SendGrid.
- Le API di backoffice forniscono piena funzionalità di gestione ordini agli operatori.
- Le operazioni critiche sono idempotenti.
- Il sistema si dimostra sicuro, scalabile e con un'ottima performance nella gestione del carico.

## 8. Rischi e punti aperti
- Rischi di sicurezza relativi all'integrazione con Stripe e la gestione delle chiavi API.
- Ambiguità nei dettagli di integrazione logistica che potrebbero influenzare il design e l'implementazione.
- Possibili problemi di timeout e comunicazione con servizi esterni potrebbero influenzare la resilienza del sistema.
- Mancanza di dettagli sui formati delle notifiche multicanale potrebbe influire sull'implementazione delle notifiche.