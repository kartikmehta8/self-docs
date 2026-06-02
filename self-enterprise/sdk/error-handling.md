# SDK error handling

The SDK throws two error classes. Catch them by type.

## `SelfApiError`

Thrown when the API returns a non-2xx response. It carries:

```ts
import { SelfApiError } from '@selfxyz/enterprise-sdk';

try {
  await self.sessions.create({ flowId, externalUuid });
} catch (err) {
  if (err instanceof SelfApiError) {
    err.statusCode;  // number (HTTP status)
    err.code;        // string ('validation_failed', 'not_found', 'unauthenticated', ...)
    err.message;     // string (human-readable)
    err.details;     // Record<string, unknown> | undefined
  } else {
    throw err;       // network error, or something unexpected
  }
}
```

### Switching on the error

Branch on `statusCode` for the HTTP-level case and on `code` for the specific reason:

```ts
try {
  await self.sessions.create({ flowId, externalUuid });
} catch (err) {
  if (!(err instanceof SelfApiError)) throw err;

  switch (err.statusCode) {
    case 400:
      return badRequest(err.details);          // details.issues lists the bad fields
    case 404:
      return notFound('That flow or session no longer exists');
    case 402:
      return paymentRequired(err.details);     // { balance, required, planTier }
    case 429:
      return tryLater();                        // back off and retry
    default:
      return serverError(err);
  }
}
```

### Error codes

`err.code` is a stable, machine-readable string. The most common from `sessions.create(...)`:

| `code` | `statusCode` | When | What to do |
| --- | --- | --- | --- |
| `validation_failed` | 400 | Request body didn't match the schema. `details.issues` lists offending fields. | Fix the request; do not retry. |
| `unauthenticated` | 401 | Missing, malformed, or revoked API key. | Check your key. |
| `unauthenticated` | 402 | Org credit balance too low (hard credit gate). `details` carries `balance`, `required`, `planTier`. | Top up or upgrade. Branch on **`statusCode` 402**, not the code. |
| `forbidden` | 403 | Key valid but not allowed for this resource (for example a test key against a live flow). | Use a key in the right environment. |
| `not_found` | 404 | `flowId` / `sessionId` doesn't exist, is archived, or belongs to another org. | Verify the ID in the dashboard. |
| `conflict` | 409 | The flow exists but has no published version. | Publish the flow, then retry. |
| `rate_limited` | 429 | Per-key rate limit exceeded. | Honor `Retry-After` and back off. |
| `vendor_unavailable` | 503 | A dependency is temporarily unavailable. | Retry with backoff. |
| `internal_error` | 500 | Server-side error. | Retry with backoff; if it persists, contact support. |

## `WebhookVerificationError`

Thrown by `SelfWebhooks.verify(...)` when the signature doesn't match the body, the timestamp is too old, or required headers are missing.

```ts
import { WebhookVerificationError } from '@selfxyz/enterprise-sdk';

try {
  const event = SelfWebhooks.verify(raw, headers, secret);
} catch (err) {
  if (err instanceof WebhookVerificationError) {
    // Respond 400; a bad signature won't get better on retry.
  }
}
```

A separate failure mode: if the signature checks out but the body doesn't match any known event schema, `verify(...)` throws a Zod `ZodError`. That usually means the server is sending a newer event type than your SDK version knows, upgrade the SDK.

## Retries

The SDK does **not** retry. A failed request throws immediately. Handle transient responses yourself: retry `429` (after `Retry-After`) and `5xx` with exponential backoff, and never retry a `4xx` other than `429`.

## Logging

Log the status and code so failures are diagnosable:

```ts
log.error({
  msg: 'self_api_error',
  statusCode: err.statusCode,
  code: err.code,
  details: err.details,
});
```

## Related

* [SDK reference](nodejs.md).
* [Verify webhooks](verify-webhooks.md).
