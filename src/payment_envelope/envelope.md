## Payment Envelope

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
	"gateway": "https://example.com/..",     // DogeConnect Gateway URL
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

Do not submit a payment transaction after the time `issued` + `timeout`.

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
