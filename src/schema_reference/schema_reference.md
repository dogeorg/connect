# Schema Reference

Quick reference for all DogeConnect JSON schemas.

All `8-DP string` fields are decimal strings with up to 8 decimal places representing Dogecoin amounts.
See [Payment Envelope](../payment_envelope/envelope.md#8-dp-string) for details.


## Enums

### EnvelopeType

| Value | Description |
|---|---|
| `payment` | Connect Payment envelope |

### ItemType

| Value | Description |
|---|---|
| `item` | Purchasable item |
| `tax` | Tax line item |
| `fee` | Fee line item |
| `shipping` | Shipping charge |
| `discount` | Discount (amount MUST be negative) |
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

<!-- ANCHOR: connect_envelope -->
```json
{
	"version": "1.0",             // MUST be 1.0
	"payload": "YTc2ZDc2MzEyZ..", // Base64-encoded JSON payload (e.g. Connect Payment)
	"pubkey": "c8a6927d0a004..",  // Relay Public Key, BIP-340 Schnorr X-only (32 bytes)
	"sig": "202d831c6437c.."     // Payload Signature, BIP-340 Schnorr (64 bytes)
}
```
<!-- ANCHOR_END: connect_envelope -->

| Field | Type | Required | Description |
|---|---|---|---|
| `version` | string | yes | Protocol version, MUST be `"1.0"` |
| `payload` | string | yes | Base64-encoded JSON payload (Connect Payment) |
| `pubkey` | string | yes | Relay public key, BIP-340 Schnorr X-only (32 bytes, hex) |
| `sig` | string | yes | Payload signature, BIP-340 Schnorr (64 bytes, hex) |


## Connect Payment

The decoded payload inside a Connect Envelope.

<!-- ANCHOR: connect_payment -->
```json
{
	"type": "payment",                       // MUST be "payment"
	"id": "PID-123",                         // Relay-unique Payment ID
	"issued": "2006-01-02T15:04:05-07:00",   // RFC 3339 Timestamp
	"timeout": 60,                           // Timeout in seconds, do not pay after this time
	"relay": "https://relay.example.com/..", // Payment Relay to submit payment tx
	"relay_token": "eyJpZCI6IlBJRC...",      // Opaque relay-generated token (optional)
	"fee_per_kb": "0.01001386",              // Min fee per 1000 bytes the relay will accept, 8-DP string
	"max_size": 10000,                       // Max tx size in bytes the relay will accept
	"vendor_icon": "https://example.com/..", // Vendor icon URL, JPG or PNG (optional)
	"vendor_name": "Vendor Co",              // Vendor display name
	"vendor_address": "123 Example St",      // Vendor business address (optional)
	"vendor_url": "https://example.com",     // Vendor website URL (optional)
	"vendor_order_url": "https://example.com/..", // URL to view order on vendor's site (optional)
	"vendor_order_id": "INV-2025-0042",      // Vendor's unique order identifier (optional)
	"order_reference": "A073",               // Short customer-facing order identifier (optional)
	"note": "Thank you for your order!",     // Free-text note from vendor to customer (optional)
	"total": "41.9395",                      // Total including fees and taxes, 8-DP string
	"fees": "1.0",                           // Fees subtotal, 8-DP string (optional)
	"taxes": "1.9495",                       // Taxes subtotal, 8-DP string (optional)
	"fiat_total": "5.00",                    // Total in fiat currency, decimal string (optional)
	"fiat_tax": "0.23",                      // Taxes in fiat currency, decimal string (optional)
	"fiat_currency": "USD",                  // ISO 4217 currency code (required with fiat_total/fiat_tax)
	"items": [],                             // List of line items to display (Connect Items)
	"outputs": []                            // List of outputs to pay (Connect Outputs)
}
```
<!-- ANCHOR_END: connect_payment -->

| Field | Type | Required | Description |
|---|---|---|---|
| `type` | string | yes | EnvelopeType enum; MUST be `"payment"`. Exists for forward compatibility with future envelope types |
| `id` | string | yes | Relay-unique payment ID |
| `issued` | string | yes | RFC 3339 timestamp |
| `timeout` | integer | yes | Timeout in seconds; do not pay after `issued + timeout` |
| `relay` | string | yes | Payment Relay base URL |
| `relay_token` | string | no | Opaque relay-generated token; wallet MUST echo in Payment Submission if present |
| `fee_per_kb` | string | yes | Minimum fee per 1000 bytes that the Payment Relay is willing to accept. Wallet MUST construct a transaction meeting at least this fee rate, 8-DP string |
| `max_size` | integer | yes | Maximum size of transaction in bytes that the Payment Relay is willing to accept |
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

<!-- ANCHOR: connect_item -->
```json
{
	"type": "item",                           // item, tax, fee, shipping, discount, donation
	"id": "SK-101",                           // unique item ID or SKU
	"icon": "https://example.com/itm/ic.png", // icon URL, JPG or PNG (optional)
	"name": "Doge Plushie",                   // name to display
	"desc": "One doge plushie in a soft bag", // item description to display (optional)
	"count": 1,                               // number of units >= 1
	"unit": "38.99",                          // unit price, 8-DP string
	"total": "38.99",                         // count x unit, 8-DP string
	"tax": "1.9495"                           // tax on this item, 8-DP string (optional)
}
```
<!-- ANCHOR_END: connect_item -->

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

<!-- ANCHOR: connect_output -->
```json
{
	"address": "DQ6dt7wCjLDxtdSwCYSAMFHwrD5Q1xybmL", // Dogecoin Address
	"amount": "1.0"                                   // Amount, 8-DP string
}
```
<!-- ANCHOR_END: connect_output -->

| Field | Type | Required | Description |
|---|---|---|---|
| `address` | string | yes | Dogecoin address |
| `amount` | string | yes | Amount to pay, 8-DP string |


## Payment Submission

The wallet's submission to the relay's `pay` endpoint in response to a Connect Payment.

<!-- ANCHOR: payment_submission -->
```json
{
	"id": "PID-123",                    // Relay-unique Payment ID from Connect Payment
	"tx": "489c47f8a3ba3293737..",      // Hex-encoded signed dogecoin transaction
	"refund": "DKY8dUTQthSX..",        // Dogecoin address for refunds (recommended)
	"relay_token": "eyJpZCI6IlBJRC..." // Relay token from Connect Payment, if present
}
```
<!-- ANCHOR_END: payment_submission -->

| Field | Type | Required | Description |
|---|---|---|---|
| `id` | string | yes | Relay-unique payment ID from Connect Payment |
| `tx` | string | yes | Hex-encoded signed Dogecoin transaction |
| `refund` | string | no | Customer-provided Dogecoin address for refunds; tx input addresses may not return funds to the customer (e.g. exchange deposits), so this provides a guaranteed return path (recommended) |
| `relay_token` | string | conditional | Opaque relay token; required when the Connect Payment contained a `relay_token` |


## Payment Status

The relay's response from both the `pay` and `status` endpoints.

<!-- ANCHOR: payment_status_accepted -->
```json
{
	"id": "PID-123",
	"status": "accepted",
	"txid": "a1b2c3d4e5..",
	"required": 5,
	"confirmed": 0,
	"due_sec": 300
}
```
<!-- ANCHOR_END: payment_status_accepted -->

<!-- ANCHOR: payment_status_declined -->
```json
{
	"id": "PID-123",
	"status": "declined",
	"reason": "Transaction deemed too risky"
}
```
<!-- ANCHOR_END: payment_status_declined -->

| Field | Type | Required | Description |
|---|---|---|---|
| `id` | string | yes | Relay-unique payment ID |
| `status` | string | yes | PaymentStatus enum |
| `reason` | string | conditional | Reason for decline; present when `declined` |
| `txid` | string | conditional | Hex-encoded tx ID; present when `accepted` or `confirmed` |
| `confirmed_at` | string | conditional | RFC 3339 timestamp of when the tx reached the required block confirmations; present when `confirmed` |
| `required` | integer | conditional | Block confirmations required; present when `accepted` or `confirmed` |
| `confirmed` | integer | conditional | Current block confirmations; present when `accepted` or `confirmed` |
| `due_sec` | integer | conditional | Estimated seconds until confirmed; present when `accepted` or `confirmed` |


## Status Query

Query the current status of a payment, submitted to the relay's `status` endpoint.

<!-- ANCHOR: status_query -->
```json
{
	"id": "PID-123" // Relay-unique Payment ID from Connect Payment
}
```
<!-- ANCHOR_END: status_query -->

| Field | Type | Required | Description |
|---|---|---|---|
| `id` | string | yes | Relay-unique payment ID from Connect Payment |


## Error Response

Returned by the relay when a request fails.

<!-- ANCHOR: error_response -->
```json
{
	"error": "invalid_tx",
	"message": "Transaction outputs do not match requested amounts"
}
```
<!-- ANCHOR_END: error_response -->

| Field | Type | Required | Description |
|---|---|---|---|
| `error` | string | yes | ErrorCode enum |
| `message` | string | yes | Human-readable error detail |
