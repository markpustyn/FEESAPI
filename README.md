```md
# Fees API

A simple JSON API for calculating total auction fees for vehicle bids.  
Supports Copart and IAAI with auction specific validation and fee logic.

---

## Overview

The Fees API calculates the full cost of a vehicle purchase based on bid amount, auction rules, and fee tiers.  
It returns a detailed breakdown including bidding fees, payment fees, and the final total.

Supported auctions:
- Copart
- IAAI

---

## Base URL

```

/api/fees

```

---

## Authentication

All requests require an API key.

**Header**
```

x-api-key: YOUR_API_KEY

```

Requests without a valid key return `401 Unauthorized`.

---

## Request

### Method
```

POST

```

### Headers
```

Content-Type: application/json
x-api-key: YOUR_API_KEY

````

### Body

```json
{
  "bidAmount": 1200,
  "auction": "copart",
  "bidType": "online",
  "bidPay": "secured",
  "bidVehicle": "standard"
}
````

### Fields

| Field      | Type   | Required | Description                                |
| ---------- | ------ | -------- | ------------------------------------------ |
| bidAmount  | number | yes      | Bid amount greater than 0                  |
| auction    | string | yes      | `copart` or `iaai`                         |
| bidType    | string | yes      | `online`, `kiosk`, `non-kiosk`             |
| bidPay     | string | yes      | `secured`, `unsecured`, `standard`, `high` |
| bidVehicle | string | yes      | Vehicle category                           |

---

## Auction Rules

### Copart

* bidType: online, kiosk, non-kiosk
* bidPay: secured, unsecured, standard, high
* bidVehicle: standard, heavy, crashedToys

### IAAI

* bidType: online, kiosk
* bidPay: standard, high
* bidVehicle: licensed, non-licensed, recRides, heavy

Invalid combinations return `400 Bad Request`.

---

## Response

### Success `200`

```json
{
  "bidAmount": 1200,
  "result": {
    "totalAmount": 1465,
    "calculatedPaymentFee": 250,
    "calculatedBiddingFee": 85
  }
}
```

### Fields

| Field                | Description            |
| -------------------- | ---------------------- |
| totalAmount          | Bid plus all fees      |
| calculatedPaymentFee | Payment processing fee |
| calculatedBiddingFee | Auction bidding fee    |

---

## Errors

### Invalid input `400`

```json
{ "error": "Invalid bidAmount" }
```

### Unsupported option `400`

```json
{
  "error": "IAAI does not support bidType \"non-kiosk\". Use \"online\" or \"kiosk\"."
}
```

### Unauthorized `401`

```json
{ "error": "Unauthorized" }
```

---

## Examples

### Copart

```bash
curl -X POST http://localhost:3000/api/fees \
  -H "content-type: application/json" \
  -H "x-api-key: YOUR_API_KEY" \
  -d '{
    "bidAmount": 1500,
    "auction": "copart",
    "bidType": "online",
    "bidPay": "secured",
    "bidVehicle": "standard"
  }'
```

### IAAI

```bash
curl -X POST http://localhost:3000/api/fees \
  -H "content-type: application/json" \
  -H "x-api-key: YOUR_API_KEY" \
  -d '{
    "bidAmount": 1500,
    "auction": "iaai",
    "bidType": "online",
    "bidPay": "standard",
    "bidVehicle": "licensed"
  }'
```

---

## Best practices

* Validate inputs on the client before calling the API
* Never expose your API key in public frontend code
* Treat the API as the single source of truth for fee totals

---

## Support

For questions or access requests, contact the API owner.

```
```
