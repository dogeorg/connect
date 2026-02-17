# Schema Reference

Quick reference for all DogeConnect JSON schemas.

All `8-DP string` fields are decimal strings with up to 8 decimal places representing Dogecoin amounts.
See [Payment Envelope](../payment_envelope/envelope.md#8-dp-string) for details.


## Enums

### PaymentType

| Value | Description |
|---|---|
| `payment` | A payment request |

### ItemType

| Value | Description |
|---|---|
| `item` | Purchasable item |
| `tax` | Tax line item |
| `fee` | Fee line item |
| `shipping` | Shipping charge |
| `discount` | Discount (negative amount) |
| `donation` | Donation |

### PaymentStatus

| Value | `pay` | `status` | Description |
|---|---|---|---|
| `unpaid` | — | yes | No transaction submitted yet |
| `accepted` | yes | yes | Transaction received, awaiting confirmations |
| `confirmed` | yes | yes | Required confirmations reached |
| `declined` | yes | — | Relay/vendor rejected the payment |

### ErrorCode

| Value | Description |
|---|---|
| `not_found` | Unknown payment ID |
| `expired` | Payment timeout has elapsed |
| `invalid_tx` | Transaction is malformed or cannot be decoded |
| `invalid_outputs` | Transaction does not pay the correct amounts/addresses |
| `invalid_token` | Relay token is missing, corrupt, or failed verification |


## Connect Envelope

Signed wrapper around a Connect Payment payload.

```json
{
	"version": "1.0",
	"payload": "YTc2ZDc2MzEyZ..",
	"pubkey": "c8a6927d0a004..",
	"sig": "202d831c6437c.."
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `version` | string | yes | Protocol version, MUST be `"1.0"` |
| `payload` | string | yes | Base64-encoded JSON payload (Connect Payment) |
| `pubkey` | string | yes | Relay public key, BIP-340 Schnorr X-only (32 bytes, hex) |
| `sig` | string | yes | Payload signature, BIP-340 Schnorr (64 bytes, hex) |


## Connect Payment

The decoded payload inside a Connect Envelope.

```json
{
	"type": "payment",
	"id": "PID-123",
	"issued": "2006-01-02T15:04:05-07:00",
	"timeout": 60,
	"relay": "https://relay.example.com/doge/v1",
	"relay_token": "eyJpZCI6IlBJRC...",
	"fee_per_kb": "0.01001386",
	"max_size": 10000,
	"vendor_icon": "https://example.com/icon.png",
	"vendor_name": "Vendor Co",
	"vendor_address": "123 Example St",
	"vendor_url": "https://example.com",
	"vendor_order_url": "https://example.com/..",
	"vendor_order_id": "INV-2025-0042",
	"order_reference": "A073",
	"note": "Thank you for your order!",
	"total": "41.9395",
	"fees": "1.0",
	"taxes": "1.9495",
	"fiat_total": "5.00",
	"fiat_tax": "0.23",
	"fiat_currency": "USD",
	"items": [],
	"outputs": []
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `type` | string | yes | PaymentType enum; MUST be `"payment"` |
| `id` | string | yes | Relay-unique payment ID |
| `issued` | string | yes | RFC 3339 timestamp |
| `timeout` | integer | yes | Timeout in seconds; do not pay after `issued + timeout` |
| `relay` | string | yes | Payment Relay base URL |
| `relay_token` | string | no | Opaque relay-generated token; wallet MUST echo in Payment Submission if present |
| `fee_per_kb` | string | yes | Minimum fee per 1000 bytes in payment tx, 8-DP string |
| `max_size` | integer | yes | Maximum size in bytes of payment tx |
| `vendor_icon` | string | no | Vendor icon URL (JPG or PNG) |
| `vendor_name` | string | yes | Vendor display name |
| `vendor_address` | string | no | Vendor business address |
| `vendor_url` | string | no | Vendor website URL |
| `vendor_order_url` | string | no | URL to view this order on vendor's site |
| `vendor_order_id` | string | no | Vendor's unique order identifier |
| `order_reference` | string | no | Short customer-facing order identifier |
| `note` | string | no | Free-text note from vendor to customer |
| `total` | string | yes | Total including fees and taxes, 8-DP string |
| `fees` | string | no | Fees subtotal, 8-DP string |
| `taxes` | string | no | Taxes subtotal, 8-DP string |
| `fiat_total` | string | no | Total in fiat currency, decimal string |
| `fiat_tax` | string | no | Taxes in fiat currency, decimal string |
| `fiat_currency` | string | conditional | ISO 4217 currency code; required when `fiat_total` or `fiat_tax` is present |
| `items` | array | yes | List of Connect Items |
| `outputs` | array | yes | List of Connect Outputs |


## Connect Item

A line item within a Connect Payment.

```json
{
	"type": "item",
	"id": "SK-101",
	"icon": "https://example.com/itm/ic.png",
	"name": "Doge Plushie",
	"desc": "One doge plushie in a soft bag",
	"count": 1,
	"unit": "38.99",
	"total": "38.99",
	"tax": "1.9495"
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `type` | string | yes | ItemType enum |
| `id` | string | yes | Unique item ID or SKU |
| `icon` | string | no | Icon URL (JPG or PNG) |
| `name` | string | yes | Display name |
| `desc` | string | no | Item description |
| `count` | integer | yes | Number of units (>= 1) |
| `unit` | string | yes | Unit price, 8-DP string |
| `total` | string | yes | count x unit, 8-DP string |
| `tax` | string | no | Tax on this item, 8-DP string |


## Connect Output

A transaction output the wallet must pay.

```json
{
	"address": "DQ6dt7wCjLDxtdSwCYSAMFHwrD5Q1xybmL",
	"amount": "1.0"
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `address` | string | yes | Dogecoin address |
| `amount` | string | yes | Amount to pay, 8-DP string |


## Payment Submission

The wallet's submission to the relay's `pay` endpoint in response to a Connect Payment.

```json
{
	"id": "PID-123",
	"tx": "489c47f8a3ba3293737..",
	"refund": "DKY8dUTQthSX..",
	"relay_token": "eyJpZCI6IlBJRC..."
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `id` | string | yes | Relay-unique payment ID from Connect Payment |
| `tx` | string | yes | Hex-encoded signed Dogecoin transaction |
| `refund` | string | no | Dogecoin address for refunds (recommended) |
| `relay_token` | string | conditional | Opaque relay token; required when the Connect Payment contained a `relay_token` |


## Payment Status

The relay's response from both the `pay` and `status` endpoints.

```json
{
	"id": "PID-123",
	"status": "accepted",
	"reason": "...",
	"txid": "a1b2c3d4e5..",
	"confirmed_at": "2006-01-02T15:04:05-07:00",
	"required": 5,
	"confirmed": 0,
	"due_sec": 300
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `id` | string | yes | Relay-unique payment ID |
| `status` | string | yes | PaymentStatus enum |
| `reason` | string | conditional | Reason for decline; present when `declined` |
| `txid` | string | conditional | Hex-encoded tx ID; present when `accepted` or `confirmed` |
| `confirmed_at` | string | conditional | RFC 3339 timestamp; present when `confirmed` |
| `required` | integer | conditional | Block confirmations required; present when `accepted` or `confirmed` |
| `confirmed` | integer | conditional | Current block confirmations; present when `accepted` or `confirmed` |
| `due_sec` | integer | conditional | Estimated seconds until confirmed; present when `accepted` or `confirmed` |


## Status Query

Query the current status of a payment, submitted to the relay's `status` endpoint.

```json
{
	"id": "PID-123"
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `id` | string | yes | Relay-unique payment ID from Connect Payment |


## Error Response

Returned by the relay when a request fails.

```json
{
	"error": "invalid_tx",
	"message": "Transaction outputs do not match requested amounts"
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `error` | string | yes | ErrorCode enum |
| `message` | string | yes | Human-readable error detail |
