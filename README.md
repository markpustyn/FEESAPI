# Auction Fees + Towing Quote API

Calculates auction fees (Copart or IAAI) and inland towing estimate to the nearest port.

## Endpoint

**POST** `https://www.auctionfeescalculator.com/api/quote`

## Authentication

Requires an API key.

**Header**
```http
x-api-key: YOUR_API_KEY
````

If missing or invalid, returns `401 Unauthorized`.

---

## Request

### Headers

```http
Content-Type: application/json
x-api-key: YOUR_API_KEY
```

### Body (JSON)

```json
{
  "bidAmount": 1200,
  "auction": "copart",
  "bidType": "online",
  "bidPay": "secured",
  "bidVehicle": "standard",
  "fromCity": "Reno",
  "fromState": "NV"
}
```

### Fields

| Field      | Type   | Required | Allowed values                                                             | Notes                                                                  |
| ---------- | ------ | -------- | -------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| bidAmount  | number | yes      | `> 0`                                                                      | Must be finite and greater than 0                                      |
| auction    | string | yes      | `copart`, `iaai`                                                           | Selects which fee calculator runs                                      |
| bidType    | string | yes      | `online`, `kiosk`, `non-kiosk`                                             | **IAAI does not support `non-kiosk`**                                  |
| bidPay     | string | yes      | `secured`, `unsecured`, `standard`, `high`                                 | **IAAI only supports `standard` or `high`**                            |
| bidVehicle | string | yes      | `standard`, `heavy`, `crashedToys`, `licensed`, `non-licensed`, `recRides` | **IAAI only supports `licensed`, `non-licensed`, `recRides`, `heavy`** |
| fromCity   | string | yes      | any non-empty string                                                       | Trimmed before use                                                     |
| fromState  | string | yes      | 2 letter state code recommended                                            | Trimmed and uppercased before use                                      |

---

## Behavior

### Copart

* bidType: **online**, kiosk, non-kiosk
* bidPay: secured, **unsecured**, standard, high

### IAAI

* bidType: **online**, kiosk
* bidPay: **standard**, high


### Towing

`towingFees` comes from `estimateInlandTowing({ fromCity, fromState })`.

If the towing function returns `"Undefined"`, the API returns:

```json
"towingFees": "Undefined"
```

Otherwise it returns the towing object directly.

### Shipping

`shippingFees` is currently a placeholder:

```json
"shippingFees": { "total": 0 }
```

---

## Response

### 200 OK

```json
{
 "bidAmount": 10400,
 "auctionFees": {
  "bidAmount": 10400,
  "auctionFees": {
   "auctionFee": 1290,
   "totalAmount": 11690
  }
 },
 "towingFees": {
  "departure": "Newburgh, NY",
  "toPort": {
   "city": "New York",
   "state": "NY"
  },
  "towingTotal": 330
 },
 "shippingFees": {
  "total": 0
 }
}
```

**Notes**

* `towingFees` can be an object **or** the string `"Undefined"`.

---

## Errors

### 401 Unauthorized

```json
{ "error": "Unauthorized" }
```

### 400 Bad Request

**Invalid JSON**

```json
{ "error": "Invalid JSON" }
```

**Invalid input**
Returned when:

* `bidAmount` is not finite or `<= 0`
* `fromCity` is missing or empty after trim
* `fromState` is missing or empty after trim

```json
{ "error": "Invalid input" }
```

**IAAI unsupported bidType**

```json
{
  "error": "IAAI does not support bidType \"non-kiosk\". Use \"online\" or \"kiosk\".",
  "bidType": "non-kiosk"
}
```

**IAAI unsupported bidPay**

```json
{
  "error": "IAAI only supports bidPay \"standard\" or \"high\".",
  "bidPay": "secured"
}
```

**IAAI unsupported bidVehicle**

```json
{
  "error": "IAAI does not support this bidVehicle.",
  "bidVehicle": "standard"
}
```

**Calculation failure**

```json
{
  "error": "Quote calculation failed",
  "details": "..."
}
```

---

## Example JavaScript

```js
await fetch("https://www.auctionfeescalculator.com/api/quote", {
        method: "POST",
        headers: {
            "Content-Type": "application/json",
            "x-api-key": process.env.CLIENT_API_KEY!,
        },
        body: JSON.stringify({
            auction: "iaai",
            bidAmount: 10400,
            bidType: "online",
            bidPay: "standard",
            bidVehicle: "licensed",
            fromCity: "Newburgh",
            fromState: "NY",
        }),
```

