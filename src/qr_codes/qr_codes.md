## Payment QR-Codes

The de-facto standard Dogecoin QR-Code encodes a URI like this:

```
dogecoin:DQ6dt7wCjLDxtdSwCYSAMFHwrD5Q1xybmL?amount=8.25
```

The URI scheme `dogecoin:` is associated with wallet software on a customer's device,
so the wallet will open when the customer scans a QR Code with the device's camera.

The above QR-Code requests a payment of 8.25 dogecoin to a specified address.
Traditional wallets simply pre-fill the amount and allow the user to broadcast
their payment transaction directly to the dogecoin network.

This payment method has some problems:
* The vendor doesn't know the customer has initiated payment until the transaction
  is mined into a block, one or more minutes later.
* The customer can choose the amount to pay, or can pay with multiple transactions,
  which is difficult for the vendor to detect and reconcile.
* The customer doesn't see what they're paying for or who the vendor is,
  before signing the transaction.

### DogeConnect QR-Codes

Due to the limited space available in a QR Code, DogeConnect addresses these
problems by including the URL of an itemised _Payment Request_.

DogeConnect adds two parameters to the QR-Code:

```
dogecoin:DQ6dt7wCjLDxtdSwCYSAMFHwrD5Q1xybmL?amount=8.25&dc=example.com%2Fdc%2Fxyz123&h=p212MS4KXZBX5uDNXWmB
```

* **dc** is a `https://` URL the wallet can use to fetch the _Payment Envelope_
* **h** is a hash of the _Payment Relay_ public key, as a security check

The `dc` URL always has the `https://` prefix trimmed off to save space in the
QR code. This URL should be kept as short as possible.

Wallets that support DogeConnect should fetch the _Payment Envelope_ from the `dc` URL
rather than using the basic dogecoin address and amount; those fields provide backwards
compatibility with older wallets.

Please note that URI parameters are _percent-encoded_ in line with RFC 3986;
wallets MUST decode the parameters according to section 2.1. We strongly recommend
using a URL/URI decoding library, which will handle all decoding requirements.
(Otherwise, make sure to use something like `decodeURIComponent` or `QueryUnescape`
functions on each parameter, _after_ splitting up the URI string.)

Wallets MUST use the `h` hash to verify that the _Payment Envelope_ came from the
vendor's nominated _Payment Relay_, as this protects against man-in-the-middle (MITM)
attacks and redirection of funds.

### Wallet Implementation

1. Receive the QR Code URI from the host system
2. Use a URL/URI decoding library to break apart the URI into a _scheme_ (`dogecoin:`),
   a _path_ component (the dogecoin address) and a series of _query parameters_:
   the `amount`, `dc` and `h` parameters.
3. Check if both `dc` and `h` are present, non-empty parameters; this indicates a
   DogeConnect payment. Both parameters must be present for a secure interaction
   (for example, a missing `h` parameter suggests a MITM attack.)
   If either are missing or empty, fall back on the _dogecoin address_ and _amount_
   parameters, i.e. treat it as a non-DogeConnect QR Code.
4. Decode the Base64-encoded `h` parameter, using a library or Base64 decode routine.
   If this fails to decode, or doesn't yield 15 bytes of data, the request is invalid
   (this also suggests a MITM attack.)
5. Concatenate `https://` onto the start of the `dc` parameter. It is always a
   _https_ URL, but the `https://` is trimmed off to keep the QR Codes from
   growing too large.
6. Fetch the DogeConnect _Payment Envelope_ JSON payload from the URL created in step 4.
   If this request fails, make a few attempts before giving up, as sometimes there
   are spurious network errors (especially on mobile devices.)
7. Decode the JSON _Payment Envelope_. If decoding fails, consider making another
   attempt (as part of the above retry process) as the response may be a spurious
   error from a network gateway; retries make the whole process more reliable.
8. Take the `pubkey` field of the _Payment Envelope_, which is hex-encoded. Use a
   hex decode routine to decode to raw bytes, verify that you decoded 32 bytes,
   then use SHA-256 to hash those bytes.
9. Compare the first 15 bytes of the hash from step 8 with the 15 bytes of the
   decoded `h` parameter from step 4. If they differ, the request is invalid and has
   likely been tampered with (this strongly suggests a MITM attack.)
10. Proceed with _Payment Envelope_ decoding and signature verification described
   in the next section.

The goals of the above process are to recognise a DogeConnect QR Code, fetch the
_Payment Envelope_ from the vendor's nominated _Payment Relay_, and verify that
the _Payment Envelope_ actually came from the nominated _Payment Relay_, by
verifying the `pubkey` hash included in the QR Code.

This protects against man-in-the-middle (MITM) attacks and redirection of funds
to a potential attacker.
