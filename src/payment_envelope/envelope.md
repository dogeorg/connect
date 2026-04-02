## Payment Envelope Schema

### Connect Envelope

{{#include ../schema_reference/schema_reference.md:connect_envelope}}

### Connect Payment

{{#include ../schema_reference/schema_reference.md:connect_payment}}

### Connect Item

{{#include ../schema_reference/schema_reference.md:connect_item}}

For items with `"type": "signing"`, the `count`, `unit`, `total`, and `tax` fields are
not applicable and MUST be omitted. A `signing` item represents a payload being
committed to the blockchain via an `OP_RETURN` output — not a purchase. The wallet
MUST display `signing` items prominently so the user understands they are signing and
broadcasting data, not just making a payment.

### Connect Output

{{#include ../schema_reference/schema_reference.md:connect_output}}

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
    If the Connect Payment contains a `relay_token`, include it in the Payment Submission.

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
