---
title: DepositGuard API

language_tabs:
  - HTTP

includes:
  - errors

search: true
---

# DepositGuard API

The DepositGuard API offers simple and secure ways to create, fund, and manage escrow accounts for vacation and rental properties.

> Example HTTP requests and responses are available below.

### Typical Flow
1.  Create an agreement.
2.  Collect payment methods.
3.  Deposit funds into escrow.
4.  Collect payout methods.
5.  Disburse security deposits and fees.

# Technical Overview

### Important Headers
* The only supported `content-type` is "application/json" and is required in every request.
* An `authorization` header is required in every request.
* A `location` header is returned any time a resource is created.  The resource identifier can be attained by parsing the url returned.

### Root URI
Use `http://api.depositguard.net` as your root.  For example, authorization would occur at `http://api.depositguard.net/oauth/token`.

### Need Access or Help?
Request credentials to the DepositGuard API or ask for assistance by emailing [api@depositguard.com](mailto:api@depositguard.com).


# Authentication

> Unlike all subsequent requests, the authentication endpoint requires the "application/x-www-form-urlencoded' content-type.

```http
POST /oauth/token_authenticated HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Authorization: Basic <Base64 encoded string of “clientId:secret”>
Host: api.depositguard.net

grant_type=client_credentials
```
``` http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "access_token": "<Access-Token>",
  "token_type": "Bearer",
  "expires_in": 28800
}
```

Authenticate your application to make requests using an OAuth2 client credentials pattern.  Your application will then be *authorized* to manage any resources (agreements, payment methods, etc.) that it creates or owns.

### Request Reference
Parameter | Type | Description
--------- | ------- | -----------
input body | String | The simple string "grant_type=client_credentials" is the only input accepted.
authorization header | String | Use HTTP basic auth for the initial request.  This is the _only_ resource that should use your clientId and secret.  To make test calls, use these credentials: </br></br> ClientId = `TestClientId` </br></br> Secret = `TestSecret`


### Response Reference
Parameter | Type | Description
--------- | ------- | -----------
access_token | String | Prepend this string with "Bearer" (the token type) and use it as the authorization header in all requests until it expires.
token_type | String | Indicates that whoever has this token is given access.  "Bearer" is the only token type supported at this time.
expires_in | Integer | Time in milleseconds until the access token expires and a new one must be attained.

<aside class="notice">
It is not possible to refresh the access token.  You must attain another when the token expires.
</aside>

# Agreements

## Create Agreement

```http
POST /agreements HTTP/1.1
Content-Type: application/json
Authorization: Bearer <Access-Token>
Host: api.depositguard.net

{
  "external_id": "123456",
  "terms": {
    "dates":{
       "start_date": "YYYY-MM-dd 12:34:56Z",
       "end_date": "YYYY-MM-dd 12:34:56Z"
    },
    "text": "terms of the agreement go here"
  },
  "renter": [{
    "first_name": "foo",
    "last_name": "bar",
    "phone": "1234567890",
    "email": "renter@gmail.com",
    "address": {
      "city": "Austin",
      "state": "TX",
      "postal_code": "78704",
      "address_line1": "123 4th Street",
      "address_line2": "Apt. 456"
    }
  }],
  "landlord":[{
    "first_name": "foo",
    "last_name": "bar",
    "phone": "1234567890",
    "email": "landlord@gmail.com",
    "address": {
      "city": "Austin",
      "state": "TX",
      "postal_code": "78704",
      "address_line1": "123 4th Street",
      "address_line2": "Apt. 456"
    }
  }]
}
```
``` http
HTTP/1.1 201 Created
Location:  http://api.depositguard.net/agreements/d646e985-b895-4fb6-b121-b221e5daa9db

```

An agreement is required before transfering funds to the DepositGuard escrow and should be communicated and agreed to by all parties involved.

All fields are required unless otherwise marked `optional`.

### Request Reference


</br>
**Root**

Parameter | Type | Description
--------- | ------- | -----------
external_id | String | `Optional.`  Can be used to match DepositGuard agreements with the integrator's systems.

</br>
**Terms**

Parameter | Type | Description
--------- | ------- | -----------
start_date | ISO 8601 string | The start date for when the property will be occupied.
end_date | ISO 8601 string | The end date for when the property will be vacated.
text | String | Should be used to record the full terms of the rental agreement.

</br>
**Renter & Landlord**

Parameter | Type | Description
--------- | ------- | -----------
first_name | String | 
last_name | String | 
phone | String | Should be stripped of dashes and periods.
email | String | Must be a valid email address.
city | String | Full city name.  US-only.
state | String | Use the ANSI 2-letter state code.  US-only.
postal_code | String | 5-digit postal code.  US-only.
address_line1 | String | 
address_line2 | String | 


## Get Agreement

```http
GET /agreements/d646e985-b895-4fb6-b121-b221e5daa9db HTTP/1.1
Content-Type: application/json
Authorization: Bearer <Access-Token>
Host: api.depositguard.net
```

``` http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "agreement_id": "d646e985-b895-4fb6-b121-b221e5daa9db",
  "external_id": "123456",
  "status": "active",
  "terms": {
    "dates":{
       "start_date": "YYYY-MM-dd 12:34:56Z",
       "end_date": "YYYY-MM-dd 12:34:56Z"
    },
    "text": "terms of the agreement go here"
  },
  "renter": [{
    "first_name": "foo",
    "last_name": "bar",
    "phone": "1234567890",
    "email": "renter@gmail.com",
    "address": {
      "city": "Austin",
      "state": "TX",
      "postal_code": "78704",
      "address_line1": "123 4th Street",
      "address_line2": "Apt. 456"
    }
  }],
  "landlord":[{
    "first_name": "foo",
    "last_name": "bar",
    "phone": "1234567890",
    "email": "landlord@gmail.com",
    "address": {
      "city": "Austin",
      "state": "TX",
      "postal_code": "78704",
      "address_line1": "123 4th Street",
      "address_line2": "Apt. 456"
    }
  }]
}
```

Fetching the agreement shows the same information used to create it, with the exception of a couple properties.

### Response Reference
Parameter | Type | Description
--------- | ------- | -----------
agreement_id | GUID | Globally unique identifier of the agreement.  Not editable.
status | String | The current status of the agreement.  Values include: </br></br>`active` - The default staus of a new agreement</br></br>`completed` - Used when a an agreement terms are satisfied and disbursement is complete</br></br>`cancelled` - Used when an agreement is no longer valid.


## Modify Agreement

```http
PUT /agreements/d646e985-b895-4fb6-b121-b221e5daa9db HTTP/1.1
Content-Type: application/json
Authorization: Bearer <Access-Token>
Host: api.depositguard.net
```

``` http
HTTP/1.1 204 No Content
```

Modifying an agreement requires a *full PUT*, meaning the entire agreement must be re-submitted if any part changes.

### Response Reference
Parameter | Type | Description
--------- | ------- | -----------
agreement_id | GUID | Globally unique identifier of the agreement.  Not editable.
status | String | The current status of the agreement.  Values include: </br></br>`active` - The default staus of a new agreement</br></br>`completed` - Used when a an agreement terms are satisfied and disbursement is complete</br></br>`cancelled` - Used when an agreement is no longer valid.



## Cancel Agreement

```http
POST /agreements/d646e985-b895-4fb6-b121-b221e5daa9db/cancel HTTP/1.1
Content-Type: application/json
Authorization: Bearer <Access-Token>
Host: api.depositguard.net

{
  "note": "renter cancelled trip."
}
```

``` http
HTTP/1.1 201 Created
Location:  http://api.depositguard.net/agreements/d646e985-b895-4fb6-b121-b221e5daa9db/history
```

Cancels the agreement and all payments and payouts.  The agreement cannot be re-activated or resumed.  This can only be performed prior to the first payment.

### Request Reference
Parameter | Type | Description
--------- | ------- | -----------
note | String | `Optional.` Leaves a note for a historical purposes.



## Complete Agreement

```http
POST /agreements/d646e985-b895-4fb6-b121-b221e5daa9db/cancel HTTP/1.1
Content-Type: application/json
Authorization: Bearer <Access-Token>
Host: api.depositguard.net

{}
```

``` http
HTTP/1.1 201 Created
Location:  http://api.depositguard.net/agreements/d646e985-b895-4fb6-b121-b221e5daa9db/complete
```

Completes the agreement.  Use this when the agreement terms are fulfilled and all parties are satisfied.

### Request Reference
Parameter | Type | Description
--------- | ------- | -----------
note | String | `Optional.` Leaves a note for a historical purposes.


## Agreement Notes

```http
POST /agreements/d646e985-b895-4fb6-b121-b221e5daa9db/cancel HTTP/1.1
Content-Type: application/json
Authorization: Bearer <Access-Token>
Host: api.depositguard.net

{
  "notes": "renter requested and was sent images of property"
}
```

``` http
HTTP/1.1 201 Created
Location:  http://api.depositguard.net/agreements/d646e985-b895-4fb6-b121-b221e5daa9db/notes
```

Add an additional note to the agreement.

### Request Reference
Parameter | Type | Description
--------- | ------- | -----------
note | String | Leaves a note for a historical purposes.



## Agreement History

```http
GET /agreements/d646e985-b895-4fb6-b121-b221e5daa9db/history HTTP/1.1
Content-Type: application/json
Authorization: Bearer <Access-Token>
Host: api.depositguard.net
```

``` http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "agreement_id": "123456",
  "events":[
    {
      "type": "agreement_created",
      "date_created": "2015-08-20T13:15:30Z"
    },
    {
      "type": "note",
      "note": "renter requested and was sent images of property",
      "date_created": "2015-08-20T13:15:30Z"
    },
    {
      "type": "agreement_modified",
      "date_created": "2015-08-23T13:15:30Z"
    },
    {
      "type": "charge",
      "note": "half of deposit",
      "date_created": "2015-08-20T13:15:30Z"
    },
    {
      "type": "charge",
      "date_created": "2015-08-20T13:15:30Z",
      "note": "half of deposit",
    },
    {
      "type": "agreement_complete",
      "date_created": "2015-08-20T13:15:30Z"
    },
    {
      "type": "disbursement",
      "note": "full disbursement",
      "date_created": "2015-08-20T13:15:30Z"
    }
  ]
}
```

Retrieve a complete history of actions related to the agreement including financial logs.

### Request Reference
Parameter | Type | Description
--------- | ------- | -----------
agreement_id | GUID | 
type | String | Kind of event.  Values include: `agreement_created`, `agreement_modified`, `agreement_complete`, `agreement_cancelled`, `deposit`, `disburse`, and `note`
note | String | Returned if any note was left.

# Payments

## Capture Payment Method

```http
POST /paymentmethods HTTP/1.1
Content-Type: application/json
Authorization: Bearer <Access-Token>
Host: api.depositguard.net

{
  "external_id": "123456",
  "number": "4111-1111-1111-1111",
  "name": "John B. Doe",
  "cvv": "123",
  "exp_month": "12",
  "exp_year": "2021",
  "city": "Austin",
  "state": "TX",
  "postal_code": "78704",
  "address_line1": "123 4th Street",
  "address_line2": "Apt. 456"
}
```
``` http
HTTP/1.1 201 Created
Location:  http://api.depositguard.net/paymentmethods/6dcbd862-e01d-4639-a5b4-774bda55ed09


```

The renter needs to provide payment information to fund the escrow account and eventually pay the landlord.  Use the external_customer_id property to connect the payment method to the user in your system.

<aside class="notice">
Payments can only be made with a credit card.</br>
</aside>

<aside class="notice">
Payment methods are not explicitly associated with a user or with an agreement.
</aside>


### Request Reference

Parameter | Type | Description
--------- | ------- | -----------
external_id | String | `Optional.`  Can be used to match DepositGuard payment methods with the integrator's systems.
number | String | 16-digit debit or credit card number with dashes.  Use `4111-1111-1111-1111` to test.
name | String |
cvv | String
exp_month | String
exp_year | String
city | String
state | String
postal_code | String
address_line1 | String
address-line2 | String


## Get Payment Method

```http
POST /paymentmethods/6dcbd862-e01d-4639-a5b4-774bda55ed09 HTTP/1.1
Content-Type: application/json
Authorization: Bearer <Access-Token>
Host: api.depositguard.net
```
``` http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "payment_method_id": "6dcbd862-e01d-4639-a5b4-774bda55ed09", 
  "external_id": "123456",
  "status": "active",
  "number": "4111-1111-1111-1111",
  "name": "John B. Doe",
  "cvv": "123",
  "exp_month": "12",
  "exp_year": "2021",
  "city": "Austin",
  "state": "TX",
  "postal_code": "78704",
  "address_line1": "123 4th Street",
  "address_line2": "Apt. 456"
}

```

Get a payment method to view its status and details.


### Response Reference
Parameter | Type | Description
--------- | ------- | -----------
payment_method_id | GUID | Globally unique identifier of the payment method.  Not editable.
status | String | The current status of the payment method.  Values include `active` and `expired`.



## Payment

```http
POST /agreements/d646e985-b895-4fb6-b121-b221e5daa9db/deposits HTTP/1.1
Content-Type: application/json
Authorization: Bearer <Access-Token>
Host: api.depositguard.net

{
  "payment_method_id": "6dcbd862-e01d-4639-a5b4-774bda55ed09",  // required
  "amount": {
    "total": "123.45",
    "currency": "USD"
  },
  "note": "half of deposit"
}
```
``` http
HTTP/1.1 201 Created
Location:  http://api.depositguard.net/agreements/d646e985-b895-4fb6-b121-b221e5daa9db/deposits/eaf557ca-ee95-4939-951a-bd2b2e1047ce

```

Authorizes and immediately charges a payment for the specified amount.  The value is held in escrow until a payout is disbursed.

### Request Reference
Parameter | Type | Description
--------- | ------- | -----------
payment_method_id | String | Payment method must be `active`.
total | String | Excludes currency symbol.
currency | String | Only `USD` is supported at this time.
note | String | Leaves a note for a historical purposes.


## Payment Status

```http
POST /agreements/d646e985-b895-4fb6-b121-b221e5daa9db/deposits/eaf557ca-ee95-4939-951a-bd2b2e1047ce HTTP/1.1
Authorization: Bearer <Access-Token>
Host: api.depositguard.net
```
``` http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "payment_method_id": "d646e985-b895-4fb6-b121-b221e5daa9db", 
  "note": "half of deposit",
  "status": "complete",
}
```

Check on the status of the payment.

### Request Reference
Parameter | Type | Description
--------- | ------- | -----------
payment_method_id | String | 
status | String | Values include `complete`, `pending`, and `failed`
note | String | Leaves a note for a historical purposes.



# Payouts

## Capture Payout Method

```http
POST /payoutmethods HTTP/1.1
Content-Type: application/json
Authorization: Bearer <Access-Token>
Host: api.depositguard.net

{
  "external_id": "123456",
  "routing_number": "123456789",
  "account_number": "951623847"
}
```
``` http
HTTP/1.1 201 Created
Location:  http://api.depositguard.net/payoutmethods/4e3acbb9-b99d-4933-8bf0-8d2fdcd11b03

```

The landlord needs to provide disbursement information for DepositGuard to payout to.  Functions the same as above, but has a different form name.


<aside class="notice">
Disbursements can only be made via ACH transfer to a bank account.
</aside>


### Request Reference

Parameter | Type | Description
--------- | ------- | -----------
external_id | String | `Optional.`  Can be used to match DepositGuard payment methods with the integrator's systems.
routing_number | String | 
account_number | String |


## Get Payout Method

```http
POST /payoutmethods/4e3acbb9-b99d-4933-8bf0-8d2fdcd11b03 HTTP/1.1
Authorization: Bearer <Access-Token>
Host: api.depositguard.net
```
``` http
HTTP/1.1 201 Created
Content-Type: application/json


{
  "external_id": "123456",
  "status":  "active",
  "routing_number": "123456789",
  "account_number": "951623847"
}
```

Get a payout method to view its status and details.

### Response Reference

Parameter | Type | Description
--------- | ------- | -----------
status | String | The current status of the payout method.  Values include `active` and `inactive`.



## Payout

```http
POST /agreements/d646e985-b895-4fb6-b121-b221e5daa9db/disbursements HTTP/1.1
Content-Type: application/json
Authorization: Bearer <Access-Token>
Host: api.depositguard.net

{
  "payout_method_id": "6dcbd862-e01d-4639-a5b4-774bda55ed09",
  "amount": {
    "total": "123.45",
    "currency": "USD"
  },
  "note": "half of deposit"
}
```
``` http
HTTP/1.1 204 No Content
Location:  http://api.depositguard.net/agreements/d646e985-b895-4fb6-b121-b221e5daa9db/disbursements/eaf557ca-ee95-4939-951a-bd2b2e1047ce

```

Disburses funds from the escrow pool.  Typically takes 3 to 7 business days.

### Request Reference
Parameter | Type | Description
--------- | ------- | -----------
payout_method_id | String | Payment method must be `active`.
total | String | Excludes currency symbol.
currency | String | Only `USD` is supported at this time.
note | String | Leaves a note for a historical purposes.



## Payout Status

```http
GET /agreements/d646e985-b895-4fb6-b121-b221e5daa9db/disbursements/eaf557ca-ee95-4939-951a-bd2b2e1047ce HTTP/1.1
Authorization: Bearer <Access-Token>
Host: api.depositguard.net
```
``` http
HTTP/1.1 200 OK
Content-Type: application/json
{
  "payout_method_id": "d646e985-b895-4fb6-b121-b221e5daa9db", 
  "note": "half of deposit",
  "status": "complete",
}
```

Disburses funds from the escrow pool.  Typically takes 3 to 7 business days.

### Response Reference
Parameter | Type | Description
--------- | ------- | -----------
payout_method_id | String | 
status | String | Values include `complete`, `pending`, and `failed`
note | String | Leaves a note for a historical purposes.