## Payment Envelope Schema

### Connect Envelope

```json
{
	"version": "1.0",             // MUST be 1.0
	"payload": "YTc2ZDc2MzEyZ..", // Base64-encoded JSON payload (e.g. ConnectPayment)
	"pubkey": "c8a6927d0a004..",  // Gateway Public Key, BIP-340 Schnorr X-only (32 bytes)
	"sig": "202d831c6437c..",     // Payload Signature, BIP-340 Schnorr (64 bytes)
}
```

### Connect Payment

```json
{
	"type": "payment",                       // MUST be "payment"
	"id": "xyz123",                          // Gateway unique payment ID
	"issued": "2006-01-02T15:04:05-07:00",   // RFC 3339 Timestamp
	"timeout": 60,                           // Timeout in seconds
	"relay": "https://example.com/..",       // Payment Relay to submit payment tx
	"vendor_icon": "https://example.com/..", // vendor icon URL, JPG or PNG
	"vendor_name": "Vendor Co",              // vendor display name
	"vendor_address": "123 Example St",      // vendor business address (optional)
	"total": "420.69",                       // Total amount including fees and taxes, DECMIAL string
	"fees": "1.0",                           // Fee subtotal, DECMIAL string
	"taxes": "5.31",                         // Taxes subtotal, DECMIAL string
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
	"cost": "414.38",                         // unit price, DECMIAL string
	"total": "414.38",                        // count x cost, DECMIAL string
}
```

### Connect Output

```json
{
	"address": "DQ6dt7wCjLDxtdSwCYSAMFHwrD5Q1xybmL", // Dogecoin Address
	"amount": "1.0",                                 // Amount, DECMIAL string
}
```

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
   A BIP-304 library will supply this step (it's essentially parsing a compressed public key.)
6. Verify the BIP-340 Schnorr signature, using the Double-SHA256 hash as the `message`, and the full Public Key and `sig`.
   A BIP-304 library will supply the signature verification algorithm.
   If this step fails, it suggests a MITM attack or faulty implementation.
7. Parse the JSON payload bytes using a standard JSON parser.
8. Check the `timeout` field: do not submit a payment transaction after the time `issued` + `timeout`.
9. Display the payment information and ask the user to confirm payment.
10. Create and sign a payment transaction and submit it to the `relay` URL.

The goals of the above process are to verify that the _Payment Envelope_ is cryptographically
signed by the _Payment Relay's_ public key, i.e. the envelope was created by the _Payment Relay_.

A reference implementation of these algorithms exist at [github.com/dogeorg/dogeconnect-go](https://github.com/dogeorg/dogeconnect-go)
which can be packaged for mobile using [gomobile bind](https://pkg.go.dev/golang.org/x/mobile/cmd/gobind).

### BIP-304 Implementations

| Language | BIP-304 Implementation                                              |
|----------|---------------------------------------------------------------------|
| C/C++    | <https://github.com/bitcoin-core/secp256k1>                         |
| Go       | <https://pkg.go.dev/github.com/btcsuite/btcd/btcec/v2/schnorr>      |
|          | <https://github.com/Yawning/secp256k1-voi>                          |
| Rust     | <https://lib.rs/crates/secp256k1>                                   |
| C#       | <https://github.com/zone117x/Secp256k1.Net>                         |
| Java     | <https://github.com/SamouraiDev/BIP340_Schnorr>                     |
|          | <https://gitlab.com/bitcoinj/secp256k1-jdk>                         |
| Dart     | <https://pub.dev/packages/bip340>                                   |
| Python   | <https://github.com/bitcoin/bips/blob/master/bip-0340/reference.py> |
