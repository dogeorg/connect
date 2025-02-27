# Payment Relay

The _Payment Relay_ base URL is found in the `relay` field of _Connect Payment_.

The following Web API must be implemented by a _Payment Relay_:

| URL                | Method | Post Data               | Response              | Function                       |
|--------------------|--------|-------------------------|-----------------------|--------------------------------|
| _relay_/**pay**    | POST   | _Payment&nbsp;Response_ | _Payment&nbsp;Reply_  | Submit&nbsp;payment&nbsp;tx    |
| _relay_/**status** | POST   | _Status&nbsp;Query_     | _Status&nbsp;Reply_   | Query&nbsp;payment&nbsp;status |

Requests and responses are JSON payloads; JSON _must_ be UTF-8 encoded.

All responses must include a `Cache-Control: no-store` header, to avoid
leaking payment information into caches. Both endpoints additionally use
the _POST_ method to avoid caching edge-cases (e.g. error responses.)

The _Relay_ **must** implement the `pay` endpoint with **idempotence** such
that, if the status of payment `id` is already `accepted` or `confirmed`,
it responds to another `pay` request with the status `accepted`, ignoring
the supplied `tx`. This is because wallets will retry their `pay` request
if they did not receive or process the first reply.


### Payment Response

This is the wallet's _response_ to the original _Payment Request_.

```json
{
    "id": "PID-123",                // Relay-unique Payment ID from Connect Payment
	"tx": "489c47f8a3ba3293737a.."  // Hex-encoded signed dogecoin transaction
}
```


### Payment Reply

This is the _Payment Relay's_ reply from the `pay` URL.

```json
{
    "id": "PID-123",       // Relay-unique Payment ID from Connect Payment
	"status": "accepted",  // One of: accepted | declined
    "reason": "",          // Reason for decline (message, optional)
    "required": 5,         // Number of block confirmations required (risk analysis)
    "confirmed": 0,        // Current number of block confirmations on-chain
    "due_sec": 300,        // Estimated time in seconds until confirmed
}
```

If the transaction is malformed, or does not pay the requested amounts to
the requested addresses, the POST will be rejected with a **400 Bad Request**
http response. This represents a _programming error_ in the wallet.

Payments may also be rejected with a `declined` status, in the case that
the _Vendor_ or their nominated _Relay_ believes the transaction is too
risky. This represents a _customer-specific_ problem.

Wallets should also be prepared to handle 500 and 503 http errors,
and other spurious errors, which indicate a temporary _Relay_ or
network problem. Wallets _should_ retry the request a few times
(with a small random delay) to increase reliability.

The final three fields, `required`, `confirmed` and `due_sec` are in common
with the _Status Reply_ below.


### Status Query

This allows the wallet to query the current status of a payment.

```json
{
    "id": "PID-123"  // Relay-unique Payment ID from Connect Payment
}
```


### Status Reply

This is the _Payment Relay's_ reply from the `status` URL.

```json
{
    "id": "PID-123",        // Relay-unique Payment ID from Connect Payment
	"status": "accepted",   // unpaid | accepted | confirmed
    "required": 5,          // Number of block confirmations required
    "confirmed": 4,         // Current number of block confirmations on-chain
    "due_sec": 30,          // Estimated time in seconds until confirmed
}
```

Payments transition from `unpaid` to `accepted` after the signed transaction
is submitted to the `pay` endpoint (provided payment was accepted.)

After the required number of block confirmations have been seen on-chain,
the payment status transitions to `confirmed`.

When the payment status is `confirmed`, the `confirmed` field is always greater
or equal to `required`, and the `due_sec` field is alway zero.

Note: there are some edge-cases where the `confirmed` count can reduce,
i.e. during a short-term blockchain fork.
