# Webhooks: overview

Webhooks are the canonical way to learn that a verification finished. Polling works but it's lossy under retries and expensive at scale.

## How they work

1. You register an HTTPS endpoint in the dashboard ([Settings → Webhooks](../dashboard/webhooks.md)). It must be live and return `2xx` to the test event to be saved, then you get the signing secret. Every endpoint receives **all** event types.
2. When an event fires, we POST a JSON payload to your endpoint, signed so you can verify it came from us.
3. Your handler verifies the signature (via the SDK) and processes the event.
4. You return `2xx` to acknowledge. A non-2xx (or timeout) triggers a retry.

```
┌──────────────┐   1. signed POST (event)     ┌──────────────────┐
│     Self     │ ───────────────────────────▶ │  Your endpoint   │
│              │                              │  verify the sig, │
│              │ ◀─────────────────────────── │  then return 2xx │
└──────────────┘   2. ack (2xx)               └──────────────────┘

No 2xx, or a timeout? Self retries the delivery with backoff.
```

## What we send

A single POST with a JSON body. The body is a discriminated-union on `type`:

| Event type | Fires when |
| --- | --- |
| `verification.completed` | Off-chain verification finishes (success or failure). |
| `verification.storage_committed` | Async storage write succeeded. |
| `verification.storage_failed` | Async storage write permanently failed. |

See [Event catalog](events.md) for the full payloads.

## Headers

Every delivery carries these signature headers:

```
svix-id: msg_2abc...               # unique message ID
svix-timestamp: 1716998400         # unix seconds
svix-signature: v1,base64...       # HMAC signature
content-type: application/json
```

On a redelivered event you may also see:

```
svix-replay: true
```

## Ordering and delivery guarantees

* **At-least-once.** The same event can arrive more than once (a retry after a failed delivery, a network blip). Make your handler idempotent.
* **No strict ordering.** `verification.completed` and `verification.storage_committed` are independent. Don't assume one arrives before the other.
* **Bounded latency.** Typically under a second from verification to delivery. Under retry backoff, latency can grow to minutes.

## Idempotency

The `svix-id` header is unique per delivery. The event itself carries a stable ID derived from the verification, so retries of the same event share it:

* `verification.completed`: `<verification_id>-completed`
* `verification.storage_committed`: `<verification_id>-storage-committed`
* `verification.storage_failed`: `<verification_id>-storage-failed`

Dedupe in your handler on this ID (or on `verification_id` plus `type`). See [Best practices](best-practices.md).

## Reliability

Failed deliveries (non-2xx, timeout, or connection error) are retried automatically on an exponential schedule over a span of days, so a brief outage on your side recovers on its own.

## Related

* [Signature verification](signature-verification.md): how to validate a delivery.
* [Event catalog](events.md): every event we send, with payloads.
* [Best practices](best-practices.md): idempotency, ordering, status handling.
* [SDK: Verify webhooks](../sdk/verify-webhooks.md): the easy path in Node.
