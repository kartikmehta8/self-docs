---
icon: triangle-exclamation
description: The error envelope, the full code catalog, and how to handle each one.
---

# Errors

Every non-2xx response uses a single envelope. The HTTP status tells you the category; the `code` tells you the specific reason.

## The envelope

```json
{
  "error": {
    "code": "validation_failed",
    "message": "Invalid request body",
    "details": {
      "issues": [
        { "path": ["externalUuid"], "message": "String must contain at least 1 character(s)" }
      ]
    }
  }
}
```

| Field | Notes |
| --- | --- |
| `code` | Stable, machine-readable. Switch on this — never on `message`. |
| `message` | Human-readable. May change; don't parse it. |
| `details` | Optional. Extra context — e.g. validation `issues`, or `balance`/`required` for credit errors. |

## Code catalog

These are the codes the API emits, with the HTTP status they come with and what to do.

| `code` | Status | Meaning | What to do |
| --- | --- | --- | --- |
| `validation_failed` | `400` | Request body failed schema validation. `details.issues` lists the offending fields. | Fix the request. **Do not retry** — it will keep failing. |
| `unauthenticated` | `401` | Missing, malformed, or invalid/revoked API key. | Check the `Authorization` header and the key. |
| `unauthenticated` | `402` | Insufficient credits to cover the session (when the credit gate is `hard`). `details` carries `balance`, `required`, `planTier`. | Top up or upgrade; see [Credits and usage](../billing/credits-and-usage.md). |
| `forbidden` | `403` | Key is valid but not allowed for this resource (e.g. test key against a live flow, or cross-org access). | Use a key in the right environment / org. |
| `not_found` | `404` | The flow or session doesn't exist, is archived, or belongs to another org. | Verify the ID in the dashboard. |
| `conflict` | `409` | The flow exists but has no published version. | Publish the flow, then retry. |
| `rate_limited` | `429` | Per-key rate limit exceeded. | Honor `Retry-After`. The SDK retries automatically. |
| `vendor_unavailable` | `503` | A dependency (e.g. the auth service) is temporarily unavailable. | Retry with backoff. |
| `internal_error` | `500` | Server-side error. | Retry with backoff. If persistent, contact support with the request ID. |

{% hint style="warning" %}
**`402` carries the `unauthenticated` code today.** Branch on the **HTTP status `402`** for the insufficient-credits case rather than on the code string, since `unauthenticated` is shared with `401`. The [SDK](../sdk/error-handling.md) exposes both `err.status` and `err.code` so you can do this cleanly.
{% endhint %}

A few codes in the catalog (`invalid_signature`, `idempotency_replay`, `webhook_test_failed`) surface on internal/webhook paths rather than the public session API; you'll rarely see them from `POST /v1/sessions`.

## Handling pattern

```ts
const res = await fetch('https://api.self.xyz/v1/sessions', {
  method: 'POST',
  headers: {
    Authorization: `Bearer ${process.env.SELF_API_KEY}`,
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({ flowId, externalUuid }),
});

if (!res.ok) {
  const { error } = await res.json();
  switch (res.status) {
    case 400: throw new BadRequest(error.details);          // fix and don't retry
    case 401:
    case 403: throw new AuthError(error.message);           // wrong/expired key
    case 402: return triggerCreditTopUp(error.details);     // out of credits
    case 404: throw new NotFound('flow not found');
    case 409: throw new Conflict('flow not published');
    case 429: return retryAfter(res.headers.get('Retry-After'));
    default:  return retryWithBackoff();                    // 5xx / vendor_unavailable
  }
}
```

In Node, the [SDK](../sdk/error-handling.md) wraps all of this in a typed `SelfApiError` with `status`, `code`, `message`, and `details` — and retries `429`/`5xx` for you.

## Retry rules

| Status | Retry? |
| --- | --- |
| `400`, `401`, `402`, `403`, `404`, `409` | **No.** These won't succeed on retry — fix the cause. |
| `429` | Yes, after `Retry-After`. |
| `500`, `503` | Yes, with exponential backoff (cap your attempts). |

## Request IDs

When a response includes a request-ID header (`X-Request-Id`), log it. Support can locate a request by ID in seconds; without one, the investigation is much slower. See [Troubleshooting](../reference/troubleshooting.md).

## Next

* [SDK error handling](../sdk/error-handling.md) — the typed `SelfApiError` and `WebhookVerificationError`.
* [Troubleshooting](../reference/troubleshooting.md) — symptom-first debugging.
