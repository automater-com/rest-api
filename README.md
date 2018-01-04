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
- 5 requests per second
- 30 requests per minute

These limits are enforced to prevent abuse and may be changed in the future without notice.
If you reach the limit server will return HTTP 429 status code and access will be blocked for next 3 minutes. 
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
##### Success response format
```
{
    "status": "success",
    "data": {
        ...
    }
}
```
##### Error response format
```
{
    "status" => "error",
    "code" => 500,
    "message" => "Invalid transaction_id param",
    "url" => "/rest/api/transactions.json"
}
```
##### List transactions
```
GET /transactions.json
```
Available *query string*:
```
limit: count of returned records (min: 1, max: 100)
page: current page of results (default: 1)
sort: sort direction (default: desc, available: asc - ascending, desc - descending)
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
  products[n][id] - id of Nth product
  products[n][quantity] - quantity of Nth product to purchase
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
         "order_currency": "USD"
    },
    "status": "success"
}
```
If you pass *connector_id* field, eg. ID of your connected PayPal account, in *payment_url* field will be returned
address to redirect your customer.
##### Post payment for transaction
```
POST /transactions/TRANSACTION_ID/payment.json
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
### Issues
If you have problem with our API please [contact us directly](https://automater.pl/en/kontakt).
