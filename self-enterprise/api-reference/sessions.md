---
icon: arrows-rotate
description: Create and read verification sessions.
---

# Sessions

A **session** is one attempt to verify one user against one [flow](../flows/anatomy.md). You create a session, hand the user its `verificationUrl`, and learn the outcome via a [webhook](../webhooks/overview.md) (or by reading the session back).

## Create a session

```
POST /v1/sessions
```

Authenticated with an API key. Creates a session against the **currently published version** of the given flow and returns a URL to send the user to.

### Request body

| Field | Type | Required | Notes |
| --- | --- | :---: | --- |
| `flowId` | string (UUID) | ✅ | The flow to verify against. Must be published and belong to your org. |
| `externalUuid` | string | ✅ | Your stable identifier for the user (1 to 128 chars). Echoed back on the session and every webhook. |
| `expiresInSeconds` | integer | No | How long the session stays openable. `60` to `86400`. Default `3600` (1 hour). |
| `metadata` | object | No | Arbitrary JSON you want attached to this session. Max 4 KB serialized. Echoed back on read. |
| `successUrl` | string (URL) | No | Where the hosted page sends the user on success. Overrides the flow default. |
| `failureUrl` | string (URL) | No | Where the hosted page sends the user on failure. Overrides the flow default. |

```bash
curl https://api.self.xyz/v1/sessions \
  -H "Authorization: Bearer $SELF_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "flowId": "9c0b4f1c-1d6c-4f1b-a8c4-9f0fa0a8d9e2",
    "externalUuid": "user_42",
    "expiresInSeconds": 3600,
    "metadata": { "campaign": "winter-2026" }
  }'
```

### Response `201`

```json
{
  "id": "7f3b2a1e-9c4d-4b2a-8e1f-2c6d5a4b3c2d",
  "externalUuid": "user_42",
  "status": "pending",
  "flowVersionId": "b1e2c3d4-5678-4abc-9def-0123456789ab",
  "verificationToken": "verify_live_p3F8xq2L9zA1bC4dE6gH8jK0",
  "verificationUrl": "https://verify.self.xyz/s/7f3b2a1e-9c4d-4b2a-8e1f-2c6d5a4b3c2d",
  "createdAt": "2026-06-01T17:33:21.412Z",
  "expiresAt": "2026-06-01T18:33:21.412Z",
  "completedAt": null
}
```

| Field | Notes |
| --- | --- |
| `id` | The session UUID. Use it to read the session, and to match webhook events (`verification_id`). |
| `status` | Always `pending` at creation. |
| `flowVersionId` | The immutable flow version this session is pinned to. Publishing a new flow version does **not** affect in-flight sessions. |
| `verificationToken` | Opaque token (`verify_<env>_…`) for the session. Most integrations don't need it directly. |
| `verificationUrl` | The URL to hand to the user. They open it in their Self app to verify. |
| `expiresAt` | After this, the session can't be completed and transitions to `expired`. |

### Errors

| Status | `code` | Cause |
| --- | --- | --- |
| `400` | `validation_failed` | Body didn't match the schema. `details.issues` lists the offending fields. |
| `401` | `unauthenticated` | Missing, malformed, or revoked API key. |
| `402` | `unauthenticated` | Insufficient credits to cover this session. `details` carries `balance`, `required`, `planTier`. (Only when your credit gate is set to `hard`.) |
| `404` | `not_found` | `flowId` doesn't exist, is archived, or belongs to another org. |
| `409` | `conflict` | The flow exists but has no published version. Publish it first. |
| `503` | `vendor_unavailable` | Auth service temporarily unavailable. Retry with backoff. |

See [Errors](errors.md) for the full catalog.

## Read a session

```
GET /v1/sessions/{id}
```

Returns the current state of a session, including the result once it completes. Useful for reconciliation or when you'd rather poll than rely solely on webhooks.

{% hint style="info" %}
**Access control:** the session UUID itself is the access token (it carries ~122 bits of entropy), so this endpoint is reachable with just the ID, the SDK still sends your API key. Treat session IDs as secrets: don't expose them in public URLs or logs you wouldn't protect.
{% endhint %}

### Response `200`

```json
{
  "id": "7f3b2a1e-9c4d-4b2a-8e1f-2c6d5a4b3c2d",
  "orgId": "a1b2c3d4-...",
  "environment": "live",
  "status": "valid",
  "createdAt": "2026-06-01T17:33:21.412Z",
  "completedAt": "2026-06-01T17:34:02.880Z",
  "expiresAt": "2026-06-01T18:33:21.412Z",
  "flowVersionId": "b1e2c3d4-5678-4abc-9def-0123456789ab",
  "externalUuid": "user_42",
  "metadata": { "campaign": "winter-2026" },
  "predicatesConfig": { "minimumAge": 18, "excludedCountries": ["USA"], "ofac": true },
  "proofAttributes": { "age_gte_18": true, "ofac_clear": true },
  "storage": {
    "state": "committed",
    "uri": "ipfs://bafy...",
    "credentialId": "cred_01H..."
  }
}
```

| Field | Type | Notes |
| --- | --- | --- |
| `status` | enum | `pending` · `valid` · `invalid` · `error` · `expired`. See [statuses](#statuses). |
| `completedAt` | string \| null | When verification finished. `null` while `pending`. |
| `predicatesConfig` | object \| null | The rules this session was evaluated against (the pinned flow version's config). |
| `proofAttributes` | object \| null | The disclosed results, predicate booleans and any explicit reveals. `null`/empty until completed with a non-error status. |
| `storage` | object | Decentralized-storage state: `state` (`pending`/`committed`/`failed`), `uri`, `credentialId`. |

### Statuses

| `status` | Meaning |
| --- | --- |
| `pending` | Created; the user hasn't completed verification yet. |
| `valid` | Proof verified and all flow predicates passed. The green-light case. |
| `invalid` | Proof verified but a predicate failed (e.g. user is 16, flow requires 18). |
| `error` | Verification failed for a technical reason (unsupported document, malformed proof). |
| `expired` | The session passed `expiresAt` before completing. |

### Errors

| Status | `code` | Cause |
| --- | --- | --- |
| `404` | `not_found` | No session with that ID. |

## Notes

* **Webhooks are the recommended path** for learning a session finished, polling is lossy and slower at scale. See [Webhooks: overview](../webhooks/overview.md). Use `GET` for reconciliation, not as your primary signal.
* **Credits** are reserved at creation against the session's per-product cost. Sessions that end `expired` or `error` are not billed. See [Credits and usage](../billing/credits-and-usage.md).

## Next

* [Flow config](flow-config.md), read a published flow's public config.
* [Errors](errors.md), the full error catalog.
* [Event catalog](../webhooks/events.md), the webhook you'll receive when a session completes.
