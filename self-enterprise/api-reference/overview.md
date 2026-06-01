---
icon: code
description: The REST API behind the Enterprise SDK, base URL, authentication, conventions.
---

# API reference: overview

The Enterprise SDK is a thin client over a small REST API. Most teams should use the [SDK](../sdk/nodejs.md), it handles auth headers, retries, and typed responses. This reference is for when you're on a language without an SDK yet, debugging on the wire, or just want to know exactly what the SDK sends.

## Base URL

```
https://api.self.xyz
```

All endpoints are versioned under `/v1`.

{% hint style="info" %}
The API host (`api.self.xyz`) is **separate** from the dashboard. Flows, API keys, webhook endpoints, and billing are managed in the [dashboard](../dashboard/overview.md), not through this API, see [What's not in the public API](#whats-not-in-the-public-api) below.
{% endhint %}

## Authentication

Authenticate with a Bearer API key in the `Authorization` header:

```
Authorization: Bearer sk_live_xxxxxxxxxxxxxxxxxxxxxx
```

* Keys come in two environments: `sk_test_…` and `sk_live_…`. The environment is baked into the key; there's no separate parameter. See [Test vs. live](../flows/test-vs-live.md).
* Issue and revoke keys under **Settings → API keys** in the dashboard. See [API keys](../dashboard/api-keys.md).
* Keys are server-side credentials. Never ship them in frontend or mobile code, session creation costs credits.

A missing or malformed header returns `401`; an invalid or revoked key returns `401`; a key used against the wrong environment's resource returns `403`. See [Errors](errors.md).

## The endpoints

| Method | Path | Auth | Purpose |
| --- | --- | --- | --- |
| `POST` | `/v1/sessions` | API key | Create a verification session. |
| `GET` | `/v1/sessions/{id}` | Session ID | Read a session's status and result. |
| `GET` | `/v1/flows/{flowId}/config` | None | Read a published flow's public config. |

Full details: [Sessions](sessions.md) and [Flow config](flow-config.md).

## Conventions

### Content type

Requests and responses are JSON. Send `Content-Type: application/json` on requests with a body.

### Identifiers

* **Session IDs** and **flow IDs** are UUIDs.
* A session also has a `verificationToken` (`verify_live_…` / `verify_test_…`), an opaque token returned alongside the ID. The `verificationUrl` you hand to users is built from the session ID.

### Timestamps

All timestamps are ISO-8601 strings in UTC (e.g. `2026-06-01T17:33:21.412Z`).

### Errors

Every error response uses one envelope:

```json
{
  "error": {
    "code": "validation_failed",
    "message": "Invalid request body",
    "details": { "issues": [/* ... */] }
  }
}
```

See the [Errors reference](errors.md) for the full code catalog and how to handle each one.

### Rate limits

Rate limits are enforced per API key. Exceeding them returns `429` with code `rate_limited`; honor the `Retry-After` header. The [SDK](../sdk/nodejs.md) retries `429` and `5xx` automatically with backoff.

### Idempotency

`POST /v1/sessions` is **not** idempotent, calling it twice creates two independent sessions (and reserves credits for each). Generate one session per verification attempt and store its ID against your user, rather than retrying creation blindly.

## What's not in the public API

The dashboard's own backend manages flows, API keys, webhook endpoints, members, and billing. Those endpoints authenticate with a dashboard login session (not an API key) and live on a different host, they are **not** part of the public, API-key-authenticated surface and aren't documented here. You manage all of them in the [dashboard](../dashboard/overview.md):

* **Flows** → [Configure a product](../dashboard/configure-a-product.md) · [Publish a flow version](../dashboard/publish-a-flow-version.md)
* **API keys** → [API keys](../dashboard/api-keys.md)
* **Webhook endpoints** → [Webhooks](../dashboard/webhooks.md)
* **Verifications (audit log)** → [Activity log](../dashboard/activity-log.md)
* **Billing** → [Billing](../dashboard/billing.md)

{% hint style="info" %}
Need programmatic flow or key management (CI provisioning, infra-as-code)? It's on the roadmap, talk to your CSM or email support@self.xyz about your use case.
{% endhint %}

## Next

* [Sessions](sessions.md), the endpoint you'll call most.
* [Flow config](flow-config.md), read a published flow's public config.
* [Errors](errors.md), the full error catalog.
* [SDK](../sdk/nodejs.md), the typed Node/TS client.
