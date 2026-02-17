# Payment Relay

The _Payment Relay_ base URL is found in the `relay` field of _Connect Payment_.

The following Web API must be implemented by a _Payment Relay_:

| URL                | Method | Post Data                   | Response                | Function                       |
|--------------------|--------|-----------------------------|-------------------------|--------------------------------|
| _relay_/**pay**    | POST   | _Payment&nbsp;Submission_   | _Payment&nbsp;Status_   | Submit&nbsp;payment&nbsp;tx    |
| _relay_/**status** | POST   | _Status&nbsp;Query_         | _Payment&nbsp;Status_   | Query&nbsp;payment&nbsp;status |

Requests and responses are JSON payloads; JSON _must_ be UTF-8 encoded.

All responses must include a `Cache-Control: no-store` header, to avoid
leaking payment information into caches. Both endpoints additionally use
the _POST_ method to avoid caching edge-cases (e.g. error responses.)

The _Relay_ **must** implement the `pay` endpoint with **idempotence** in
the following way: if the status of payment `id` is already `accepted` or
`confirmed`, it responds to another `pay` request with that status, ignoring
the supplied `tx`. This is because wallets will retry their `pay` request
if they did not receive or process the first reply.


### Relay Token

A _Relay_ may include an opaque `relay_token` in the _Connect Payment_. If present,
the wallet **must** echo it back verbatim in the _Payment Submission_. The token is
opaque to the wallet — it should not attempt to parse or interpret its contents.

The relay token enables **stateless validation**: the relay can embed whatever
parameters it needs inside the token, avoiding additional lookups at submission time.

The token format is entirely up to the relay implementation.


### Payment Submission

The wallet's submission to the relay's `pay` endpoint in response to a _Connect Payment_.

```json
{
	"id": "PID-123",                    // Relay-unique Payment ID from Connect Payment
	"tx": "489c47f8a3ba3293737..",  // Hex-encoded signed dogecoin transaction
	"refund": "DKY8dUTQthSX..",     // Dogecoin address for refunds (RECOMMENDED)
	"relay_token": "eyJpZCI6IlBJRC...", // Relay token from Connect Payment, if present
}
```


### Payment Status

Both the `pay` and `status` endpoints return a _Payment Status_ response.

**200 — Accepted:**

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

**403 — Declined:**

```json
{
	"id": "PID-123",
	"status": "declined",
	"reason": "Transaction deemed too risky"
}
```

The `status` field is a _PaymentStatus_ enum value. The `txid` field
contains the hex-encoded transaction ID and is present whenever the
status is `accepted` or `confirmed`. The `confirmed_at` field is an
RFC 3339 timestamp and is only present when the status is `confirmed`.

The status will be `accepted` if the _Relay_ requires one or more block
confirmations on the blockchain, reflected in the `required` field.
The status may be `confirmed` if the _Relay_ deems the payment low-risk.

The `required`, `confirmed` and `due_sec` fields are present whenever the
status is `accepted` or `confirmed`. The `required` field indicates how many
block confirmations the _Relay_ requires; `confirmed` is the current count
on-chain; `due_sec` is the estimated seconds remaining until `confirmed`.

When the payment status is `confirmed`, the `confirmed` field is always
greater or equal to `required`, and the `due_sec` field is always zero.

Payments transition from `unpaid` to `accepted` after the signed transaction
is submitted to the `pay` endpoint (provided payment was accepted.) After
the required number of block confirmations have been seen on-chain, the
payment status transitions to `confirmed`.

Note: there are some edge-cases where the `confirmed` count can reduce,
i.e. during a short-term blockchain fork.


### Error Response

```json
{
	"error": "invalid_tx",
	"message": "Transaction outputs do not match requested amounts"
}
```


### HTTP Status Codes

#### `pay` endpoint

| HTTP | Body | Meaning |
|---|---|---|
| 200 | Payment Status | Accepted or confirmed |
| 400 | Error Response | Programming error (malformed tx, wrong outputs, expired, invalid token) |
| 403 | Payment Status | Declined (`status: "declined"`) |
| 404 | Error Response | Unknown payment ID |
| 500 / 503 | - | Transient server error; retry with backoff |

#### `status` endpoint

| HTTP | Body | Meaning |
|---|---|---|
| 200 | Payment Status | Status returned |
| 404 | Error Response | Unknown payment ID |
| 500 / 503 | - | Transient server error; retry with backoff |

A **400** or **403** response indicates a permanent failure; wallets should
not retry. A **500** or **503** response is transient — the wallet should
assume the relay will recover and retry a few times with exponential backoff
and a small random jitter.


### Status Query

This allows the wallet to query the current status of a payment.

```json
{
	"id": "PID-123"  // Relay-unique Payment ID from Connect Payment
}
```
