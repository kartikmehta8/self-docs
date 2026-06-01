# SDK error handling

The SDK throws two error classes. Catch them by type.

## `SelfApiError`

Thrown when the Enterprise API returns a non-2xx response.

```ts
import { SelfApiError } from '@selfxyz/enterprise-sdk';

try {
  await self.sessions.create({ flowId, externalUuid });
} catch (err) {
  if (err instanceof SelfApiError) {
    err.status;     // number (HTTP status)
    err.code;       // string ('validation_failed', 'flow_not_found', ...)
    err.message;    // string (human-readable)
    err.details;    // Record<string, unknown> | undefined
  } else {
    throw err;      // network error, unexpected
  }
}
```

### Switching on `code`

```ts
import { SelfApiError } from '@selfxyz/enterprise-sdk';

try {
  await self.sessions.create({ flowId, externalUuid });
} catch (err) {
  if (!(err instanceof SelfApiError)) throw err;

  // Branch on `status` for HTTP-level cases (it's the most stable signal),
  // and on `code` for the specific reason.
  switch (err.status) {
    case 400:
      // err.details.issues is a Zod-issue array
      return badRequest(err.details);

    case 404:
      return notFound('That flow or session no longer exists');

    case 402:
      // Insufficient credits — trigger top-up flow / alert ops
      return paymentRequired(err.details);   // { balance, required, planTier }

    case 429:
      // SDK already retried; we're out of budget
      return tryLater();

    default:
      return serverError(err);
  }
}
```

### Error codes

`err.code` is one of the API's canonical [error codes](../api-reference/errors.md). The most common from `sessions.create(...)`:

| `code` | `status` | When | What to do |
| --- | --- | --- | --- |
| `validation_failed` | 400 | Request body didn't match the schema. `details.issues` lists offending fields. | Fix the request; do not retry. |
| `unauthenticated` | 401 | Missing, malformed, or revoked API key. | Check your key. |
| `unauthenticated` | 402 | Org credit balance too low (hard credit gate). `details` carries `balance`, `required`, `planTier`. | Top up or upgrade; branch on **status `402`**, not the code. |
| `forbidden` | 403 | Key valid but not allowed for this resource (e.g. test key + live flow, cross-org session). | Use a key in the right environment / org. |
| `not_found` | 404 | `flowId`/`sessionId` doesn't exist, is archived, or belongs to another org. | Verify the ID in the dashboard. |
| `conflict` | 409 | The flow exists but has no published version. | Publish the flow, then retry. |
| `rate_limited` | 429 | Per-key rate limit exceeded. | Honor `Retry-After`. The SDK already retries with backoff. |
| `vendor_unavailable` | 503 | A dependency is temporarily unavailable. | Retry with backoff. |
| `internal_error` | 500 | Server-side error. | Retry with backoff. If persistent, contact support with the request ID. |

See the [full error reference](../api-reference/errors.md) for the complete catalog.

## `WebhookVerificationError`

Thrown by `SelfWebhooks.verify(...)` when the signature doesn't match the body, the timestamp is too old, or required headers are missing.

```ts
import { WebhookVerificationError } from '@selfxyz/enterprise-sdk';

try {
  const event = SelfWebhooks.verify(raw, headers, secret);
} catch (err) {
  if (err instanceof WebhookVerificationError) {
    // Respond 400. We won't retry a 4xx, which is what you want for a bad signature.
  }
}
```

A separate failure mode: if the signature checks out but the body shape doesn't match any known event schema, `verify(...)` throws a Zod `ZodError`. That usually means you're on an old SDK version and the server is sending a newer event type, upgrade the SDK.

## Retrying

The SDK already retries internally on transient errors:

* `429 rate_limited`: exponential backoff honoring `Retry-After`.
* `5xx`: exponential backoff, capped at 5 attempts.

You don't need to wrap calls in your own retry loop. If you do, retry only on these statuses, never on 4xx.

## Logging

`SelfApiError` carries a `requestId` field when the server included `X-Request-Id`. Log it:

```ts
log.error({
  msg: 'self_api_error',
  code: err.code,
  status: err.status,
  requestId: err.requestId,
  details: err.details,
});
```

Support can find a request by `requestId` in seconds; without it, much slower.

## Related

* [Webhook signature verification](../webhooks/signature-verification.md).
* [Best practices](../webhooks/best-practices.md).
