# Automater RESTful API
This repository contains Automater's RESTful API documentation. All requests must be made using secure HTTPS connection to the endpoint:
```
https://automater.pl/rest/api
```
### Prerequisites
First you need API key and API secret - you can generate it by login to Automater, next going to "Settings / settings" tab and select "API" from left-side menu.

It is recommended to use one of our SDK libraries: [PHP SDK](https://github.com/automater-pl/rest-php-sdk).
### API usage limits
Automater API has the following limits:
- 10 requests per second
- 60 requests per minute

These limits are enforced to prevent abuse and may be changed in the future without notice.
If you reach the limit server will return HTTP 429 status code and access will be blocked for next 3 minutes. 
### Webhooks
Automater API allows you to receive webhooks about specified events (e.g. new transaction). Notifications can be sent as POST request to specified URL.
You can enable them by going to "Settings / settings" tab and select "API" from left-side menu.

All requests are sent as JSON body. Each time when you receive notification you should verify *X-Notification-Secret* header 
and compare it to *notification key* provided in the webhook settings to confirm its origin.

After receiving notification your server should return one of HTTP success status codes (200, 201, 202, 204). In case of any other response Automater will retry notification delivery up to 5 days with random frequency.

[Check examples with request headers and body structure.](https://github.com/automater-pl/rest-api/blob/master/webhook-requests.md)
### Security
Each request should have ***X-Api-Key*** header with your *API Key*.
In case of using POST or PUT methods you need to calculate *signature* string and pass it in ***X-Api-Signature*** header. 
If passed key or signature is invalid HTTP 401 status codes will be returned.

Below is instruction how to calculate ***X-Api-Signature*** for POST or PUT method.
1. Sort all of passed params alphabetically - please note that you must sort also nested elements.
2. Create *query string* from passed params. ***Do not url-encode string***. For example "test!" should be passed to hashing function as "test!", not as "test%21".
3. Add *API Secret* key to end of received string.
4. Hash string using SHA-256 method.

For example if you'd like to sign below packet:
```
[
  transaction_id => 1,
  email => 'example@domain.com',
  products => [
    [
      id => 7,
      quantity => 1
    ]
  ]
]
```
You should sort it and create *query string* as above:
```
email=example@domain.com&products[0][id]=7&products[0][quantity]=1&transaction_id=1
```
At the end you need to hash received ***query string*** with *API Secret* key:
```
SHA256(email=example@domain.com&products[0][id]=7&products[0][quantity]=1&transaction_id=1YOUR_API_SECRET_HERE)
```

### Requests
All responses are returned as JSON. In case of positive result server will respond with success HTTP 2xx status code.
POST and PUT requests should be send with request body as *application/x-www-form-urlencoded* (form fields).
#### Success response format
```
{
    "status": "success",
    "data": {
        ...
    }
}
```
#### Error response format
```
{
    "status" => "error",
    "code" => 500,
    "message" => "Invalid transaction_id param",
    "url" => "/rest/api/transactions.json"
}
```
#### Available requests
1. Transactions
   - [List transactions](#list-transactions)
   - [Create transaction](#create-transaction)
   - [Post payment for transaction](#post-payment-for-transaction)
   - [Add note to registry](#add-note-to-registry)
2. Products
   - [List products](#list-products)
   - [Get product details](#get-product-details)
3. Databases
   - [List databases](#list-databases)
   - [Create database](#create-database)
4. Codes
   - [Add codes to database](#add-codes-to-database)
##### List transactions
```
GET /transactions.json
```
Available *query string*:
```
limit: count of returned records (min: 1, max: 100)
page: current page of results (default: 1)
sort: sort direction (default: desc, available: asc - ascending, desc - descending)
cart_id: list transactions from specific cart
```
Approximate server response:
```
{
    "data": {
        "_currentPage": 2,
        "_pagesCount": 14,
        "_recordsCount": 14,
        "_currentCount": 1,
        "transactions": [
            {
                "id": 356232224,
                "product_id": 2242142,
                "paid": true,
                "mail": "test@domain.com",
                "phone": "",
                "quantity": 1,
                "price": 1,
                "currency": "PLN",
                "sent": 1,
                "created": 1514728514
            }
        ]
    },
    "status": "success"
}
```
##### List products
```
GET /products.json
```
Available *query string*:
```
limit: count of returned records (min: 1, max: 100)
page: current page of results (default: 1)
type: type of listings (default: all, available: 1 - Allegro, 2 - products, 3 - eBay)
status: status of listings (default: all, available: 1 - active, 2 - inactive)
```
Approximate server response:
```
{
    "data": {
        "_currentPage": 2,
        "_pagesCount": 14,
        "_recordsCount": 14,
        "_currentCount": 1,
        "products": [
            {
                "id": 38,
                "status": 1,
                "type": 2,
                "name": "TEST PRODUCT",
                "ext_offer_id": 7123664374,
                "url": "https://automater.pl/purchase/38/test-product",
                "description": "<p>test-product</p>",
                "price": 1,
                "currency": "PLN",
                "available_codes": 181,
                "database_id": 480
            }
        ]
    },
    "status": "success"
}
```
##### Get product details
```
GET /products/{PRODUCT_ID}.json
```
You should change {PRODUCT_ID} in request URI to your product ID. Approximate server response:
```
{
    "data": {
        "product": {
            "id": 3245323324,
            "status": 1,
            "type": 2,
            "name": "test bonusy",
            "url": "https://automater.pl/purchase/3245323324/test-product",
            "description": "<p>test product</p>",
            "price": 1,
            "currency": "USD",
            "available_codes": 4,
            "database_id": 32442353
        }
    },
    "status": "success"
}
```
##### Create transaction
```
POST /transactions.json
```
Available *form fields*:
```
email: customer e-mail address (required)
phone: customer phone number (default: empty)
language: customer language (default: en, available: pl - polish, en - english)
connector_id: id of payment gateway if you'd like to redirect customer to payment page, eg. PayPal (default: empty)
products: array of purchased products in below format (required):
  products[0][id] - id of first product
  products[0][quantity] - quantity of first product to purchase
  products[0][price] - price of first product (if not passed value will be taken from product settings)
  products[0][currency] - currency of first product (if not passed value will be taken from product settings)
  products[n][id] - id of Nth product
  products[n][quantity] - quantity of Nth product to purchase
  products[n][price] - price of Nth product (if not passed value will be taken from product settings)
  products[n][currency] - currency of Nth product (if not passed value will be taken from product settings)
send_status_email: switch to send e-mail with order status (default: y, available: y - true, n - false)
custom: custom string to add to transacation log (default: empty, available: 255 chars custom string)
```
Approximate server response:
```
{
    "data": {
         "cart_id": 3242,
         "created_at": 15477276362,
         "payment_url": null,
         "order_amount": 11.32,
         "order_currency": "USD",
         "status_url": "https://automater.pl/cart-status/3424/dsknfkjbshjafhec3u78bfdw/"
    },
    "status": "success"
}
```
If you pass *connector_id* field, eg. ID of your connected PayPal account, in *payment_url* field will be returned
address to redirect your customer.
##### Post payment for transaction
```
POST /transactions/CART_ID/payment.json
```
Available *form fields*:
```
payment_id: your ID of payment, should be unique (required)
amount: amount of payment (must be the same or greater than purchase amount) (required)
currency: currency of payment (must be the same as purchase currency) (required)
description: optional description of payment (default: empty, available: 100 chars custom string)
custom: custom string to add to transacation log (default: empty, available: 255 chars custom string)
```
Approximate server response:
```
{
    "data": {
        "cart_id" => 4234,
        "payment_id" => "JDNSJ2RFJNW",
        "amount" => 12.32,
        "currency" => "USD"
    },
    "status": "success"
}
```
##### Add note to registry
```
POST /transactions/TRANSACTION_ID/note.json
```
Available *form fields*:
```
note: note which should be added to registry, max 255 chars (required)
```
Approximate server response:
```
{
    "data": {},
    "status": "success"
}
```
##### List databases
```
GET /databases.json
```
Available *query string*:
```
limit: count of returned records (min: 1, max: 100)
page: current page of results (default: 1)
type: type of databases (default: all, available: 1 - standard, 2 - resursive)
```
Approximate server response:
```
{
    "data": {
        "_currentPage": 2,
        "_pagesCount": 14,
        "_recordsCount": 14,
        "_currentCount": 1,
        "databases": [
            {
                "id": 38,
                "type": 1, (1 - standard, 2 - resursive)
                "name": "test_database",
                "codes_available": 14, (count of not used codes) 
                "codes_sent": 3 (count of sent / used codes)
            }
        ]
    },
    "status": "success"
}
```
##### Create database
```
POST /databases.json
```
Available *form fields*:
```
type: type of base - 1: standard, 2: recursive (required)
name: name of base (available: 100 chars custom string, required)
```
Approximate server response:
```
{
    "data": {
        "id" => 36 (id of newly created base)
    },
    "status": "success"
}
```
##### Add codes to database
```
POST /codes/DATABASE_ID.json
```
Available *form fields*:
```
codes: array of string with codes to add (required), e.g.:
  codes[0] = "test_code"
```
Approximate server response:
```
{
    "data": {
        "database_id" => 36, (id of base to which the codes were added)
        "added_count" => 17 (count of added codes)
    },
    "status": "success"
}
```
### Issues
If you have problem with our API please [contact us directly](https://automater.pl/kontakt).
