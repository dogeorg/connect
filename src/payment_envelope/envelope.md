## Payment Envelope Schema

### Connect Envelope

```json
{
	"version": "1.0",             // MUST be 1.0
	"payload": "YTc2ZDc2MzEyZ..", // Base64-encoded JSON payload (e.g. Connect Payment)
	"pubkey": "c8a6927d0a004..",  // Relay Public Key, BIP-340 Schnorr X-only (32 bytes)
	"sig": "202d831c6437c..",     // Payload Signature, BIP-340 Schnorr (64 bytes)
}
```

### Connect Payment

```json
{
	"type": "payment",                       // MUST be "payment"
	"id": "PID-123",                         // Relay-unique Payment ID
	"issued": "2006-01-02T15:04:05-07:00",   // RFC 3339 Timestamp
	"timeout": 60,                           // Timeout in seconds, do not pay after this time
	"relay": "https://example.com/..",       // Payment Relay to submit payment tx
    "fee_per_kb": "0.01001386",              // Minimum fee per 1000 bytes in payment tx, 8-DP string
    "max_size": 10000,                       // Maximum size in bytes of payment tx
	"vendor_icon": "https://example.com/..", // Vendor icon URL, JPG or PNG
	"vendor_name": "Vendor Co",              // Vendor display name
	"vendor_address": "123 Example St",      // Vendor business address (optional)
	"total": "41.9395",                      // Total including fees and taxes, 8-DP string
	"fees": "1.0",                           // Fees subtotal, 8-DP string
	"taxes": "1.9495",                       // Taxes subtotal, 8-DP string
	"fiat_total": "5.00",                    // Total in fiat currency, decimal string (optional)
	"fiat_tax": "0.23",                      // Taxes in fiat currency, decimal string (optional)
    "fiat_currency": "USD",                  // ISO 4217 currency code (required with fiat_total/fiat_tax)
	"items": [],                             // List of line items to display (Connect Items)
	"outputs": [],                           // List of outputs to pay (Connect Outputs)
}
```

### Connect Item

```json
{
    "type": "item",                           // item, tax, fee, shipping, discount, donation
	"id": "SK-101",                           // unique item ID or SKU
	"icon": "https://example.com/itm/ic.png", // icon URL, JPG or PNG
	"name": "Doge Plushie",                   // name to display
	"desc": "One doge plushie in a soft bag", // item description to display
	"count": 1,                               // number of units >= 1
	"unit": "38.99",                          // unit price, 8-DP string
	"total": "38.99",                         // count x unit, 8-DP string
	"tax": "1.9495",                          // tax on this item, 8-DP string (optional)
}
```

### Connect Output

```json
{
	"address": "DQ6dt7wCjLDxtdSwCYSAMFHwrD5Q1xybmL", // Dogecoin Address
	"amount": "1.0",                                 // Amount, 8-DP string
}
```

### 8-DP string

Fields marked `8-DP string` are DECIMAL numbers with up to 8 decimal places,
which represent a Koinu value – the smallest unit of Dogecoin.

They are strings to preserve accuracy; most JSON parsers treat numbers as
_floating point_, which sacrifices some accuracy (e.g. a value of "0.1" cannot
be stored _accurately_ as a _floating point_ number.)

These can be parsed using a _Decimal_ library, or code like [this](https://github.com/dogeorg/dogeconnect-go/blob/main/koinu/parse.go).


## Payment Envelope Verification

The DogeConnect _Payment Envelope_ contains a Base64-encoded JSON payload,
the _Payment Relay's_ public key, and a digital signature of the payload
signed by the _Payment Relay's_ private key.

Use the following steps to decode and verify the payload.

1. Ensure you verify the `pubkey` hash as described in the [Payment QR-Code](../qr_codes/qr_codes.md) section.
2. Decode the Base64-encoded `payload` field, yielding bytes.
3. Decode the Hex-encoded `pubkey` and `sig` fields, yielding bytes.
4. Compute the Double-SHA256 of the decoded payload bytes from step 2, i.e. `SHA-256(SHA-256(bytes))`
5. Apply BIP-340 `lift_x` algorithm on the `pubkey` bytes to recover the full Public Key.
   A BIP-340 library will supply this step (it's essentially parsing a compressed public key.)
6. Verify the BIP-340 Schnorr signature, using the Double-SHA256 hash as the `message`, and the full Public Key and `sig`.
   A BIP-340 library will supply the signature verification algorithm.
   If this step fails, it suggests a MITM attack or faulty implementation.
7. Parse the JSON payload bytes using a standard JSON parser.
8. Check the `timeout` field: do not submit a payment transaction after the time `issued` + `timeout`.
9. Display the payment information and ask the user to confirm payment.
10. Create and sign a payment transaction and submit it to the [Payment Relay](../payment_relay/relay.md).

The goals of the above process are to verify that the _Payment Envelope_ is cryptographically
signed by the _Payment Relay's_ private key, i.e. the envelope was created by the _Payment Relay_.

A reference implementation of these algorithms exist at [github.com/dogeorg/dogeconnect-go](https://github.com/dogeorg/dogeconnect-go)
which can be packaged for mobile using [gomobile bind](https://pkg.go.dev/golang.org/x/mobile/cmd/gobind).

### BIP-340 Implementations

| Language | BIP-340 Implementation                                              |
|----------|---------------------------------------------------------------------|
| C/C++    | <https://github.com/bitcoin-core/secp256k1>                         |
| Go       | <https://pkg.go.dev/github.com/btcsuite/btcd/btcec/v2/schnorr>      |
|          | <https://github.com/Yawning/secp256k1-voi>                          |
| Rust     | <https://lib.rs/crates/secp256k1>                                   |
| C#       | <https://github.com/zone117x/Secp256k1.Net>                         |
| Java     | <https://github.com/SamouraiDev/BIP340_Schnorr>                     |
| Dart     | <https://pub.dev/packages/bip340>                                   |
| Python   | <https://github.com/bitcoin/bips/blob/master/bip-0340/reference.py> |
