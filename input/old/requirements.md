# Requisiti Tecnici e Funzionali -- Web Application E-Commerce Multi-Prodotto

## 1. Obiettivo del Progetto

Realizzazione di una web application e-commerce multi-prodotto per la
vendita online di prodotti appartenenti a diverse categorie.\
L'applicazione deve consentire:

-   Consultazione catalogo prodotti
-   Autenticazione utenti
-   Gestione carrello
-   Checkout e pagamento
-   Gestione e consultazione ordini

------------------------------------------------------------------------

## 2. Stack Tecnologico

### Frontend

-   Angular (versione stabile recente)
-   NodeJS
-   TypeScript
-   Angular Router
-   Reactive Forms
-   HttpClient

### Backend

-   Java 17+
-   Spring Boot
-   Spring Web
-   Spring Data JPA
-   Spring Security
-   JWT
-   BCrypt

### Database

-   PostgreSQL

### Integrazione

-   REST API JSON
-   Autenticazione stateless con JWT
-   CORS configurato

------------------------------------------------------------------------

## 3. Requisiti Funzionali

### 3.1 Catalogo Prodotti

-   Visualizzazione lista prodotti paginata
-   Visualizzazione dettaglio prodotto
-   Filtri per categoria
-   Filtri per range di prezzo
-   Ricerca per nome o descrizione
-   Visualizzazione immagini, prezzo, disponibilità

### 3.2 Gestione Utenti

-   Registrazione con email univoca
-   Password minimo 6 caratteri
-   Login con generazione JWT
-   Password salvate con BCrypt
-   Gestione errori autenticazione

### 3.3 Carrello

-   Aggiunta prodotto al carrello
-   Aggiornamento quantità
-   Rimozione prodotto
-   Calcolo totale ordine

### 3.4 Checkout e Ordini

-   Inserimento indirizzo spedizione
-   Simulazione pagamento (oppure integrazione provider)
-   Creazione ordine con ID univoco
-   Storico ordini utente

------------------------------------------------------------------------

## 4. Requisiti Tecnici Backend

-   Architettura a livelli (Controller / Service / Repository / DTO /
    Security)
-   Spring Security con filtro JWT
-   Bean Validation
-   Gestione errori centralizzata
-   Logging base
-   Codici HTTP standard: 200, 400, 401, 404, 500

------------------------------------------------------------------------

## 5. Modello Dati

### User

-   id
-   email
-   password
-   role

### Product

-   id
-   sku
-   name
-   description
-   price
-   currency
-   imageUrl
-   category
-   stockQty
-   isActive

### Category

-   id
-   name

### Cart

-   id
-   user

### CartItem

-   id
-   cart
-   product
-   quantity

### Order

-   id
-   user
-   totalAmount
-   status
-   createdAt

### OrderItem

-   id
-   order
-   product
-   quantity
-   price

------------------------------------------------------------------------

## 6. Requisiti Frontend

-   Routing Angular (catalogo, dettaglio, login, registrazione,
    carrello, checkout, ordini)
-   Reactive Forms
-   HTTP Interceptor per JWT
-   Auth Guard
-   UI responsive

------------------------------------------------------------------------

## 7. API REST Principali

### Autenticazione

-   POST /api/auth/register
-   POST /api/auth/login

### Catalogo

-   GET /api/products
-   GET /api/products/{id}
-   GET /api/categories

### Carrello

-   GET /api/cart
-   POST /api/cart/items
-   PUT /api/cart/items/{itemId}
-   DELETE /api/cart/items/{itemId}

### Ordini

-   POST /api/orders/checkout
-   GET /api/orders
-   GET /api/orders/{id}

------------------------------------------------------------------------

## 8. Sicurezza

-   JWT stateless
-   Password BCrypt
-   CORS configurato
-   Validazione input server-side
-   Nessuna credenziale hardcoded

------------------------------------------------------------------------

## 9. Non Functional Requirements

-   Tempo risposta API target \~1 secondo
-   Codice conforme best practice
-   Deploy locale supportato
-   Architettura estendibile e scalabile
