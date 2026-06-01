---
icon: sliders
description: Read a published flow's public configuration.
---

# Flow config

```
GET /v1/flows/{flowId}/config
```

Returns the **public** configuration of a flow's currently published version. No authentication required — this endpoint exposes only what a verification page needs to render (the predicate rules), never anything org-private.

It's used by the hosted verification page, and is handy when you render your own QR / verification UI and need to know what a flow asks for without creating a session.

## Path parameters

| Parameter | Type | Notes |
| --- | --- | --- |
| `flowId` | string (UUID) | The flow to read. Must be published and not archived. |

## Request

```bash
curl https://api.self.xyz/v1/flows/9c0b4f1c-1d6c-4f1b-a8c4-9f0fa0a8d9e2/config
```

## Response `200`

```json
{
  "flowId": "9c0b4f1c-1d6c-4f1b-a8c4-9f0fa0a8d9e2",
  "orgId": "a1b2c3d4-5678-4abc-9def-0123456789ab",
  "predicatesConfig": {
    "minimumAge": 18,
    "excludedCountries": ["USA"],
    "ofac": true
  }
}
```

| Field | Type | Notes |
| --- | --- | --- |
| `flowId` | string | The flow's ID. |
| `orgId` | string | The owning org's ID. |
| `predicatesConfig` | object \| null | The published version's rule set — the same predicates a user will be asked to prove. Shape depends on the flow's product; see [Anatomy of a flow](../flows/anatomy.md) and [Disclosures](../flows/disclosures.md). |

## Errors

| Status | `code` | Cause |
| --- | --- | --- |
| `404` | `not_found` | No such flow, the flow is archived, or it has no published version. |

## Notes

* This returns the **latest published** version's config. There's no way to read a draft or a historical version through this endpoint — drafts live only in the dashboard, and historical versions are pinned per session (`flowVersionId`).
* To act on a verification result, create a [session](sessions.md) and handle its [webhook](../webhooks/events.md). This endpoint is read-only metadata.

## Next

* [Sessions](sessions.md) — create a verification session.
* [Anatomy of a flow](../flows/anatomy.md) — what's in a flow config.
