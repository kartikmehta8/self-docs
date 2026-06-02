# Anatomy of a flow

A **flow** is a published, versioned configuration of one product. This page is the data model behind the dashboard's Configure tab.

## Product first

Every flow has a product, chosen when you create it. The product decides what the user proves and which rules you can set:

* **Pre KYC**: document, optional age floor, country rules, OFAC.
* **Age Verification**: an age threshold.
* **Proof of Human**: unique humanity, no extra rules.

See [Configure a product](../dashboard/configure-a-product.md) for the full breakdown.

## Three layers

```
Flow
├─ Rules        what the user must prove (depends on the product)
├─ Documents    which credentials they can prove it with
└─ Settings     operational metadata (URLs, branding)
```

Each layer is independently configurable. They combine into a **flow version** when you publish.

## Rules

Rules are predicates over identity attributes. The user proves each predicate without revealing the underlying value. The available rules come from the product. The published config is a small JSON object. For a Pre KYC or Age Verification flow it looks like:

```json
{
  "minimumAge": 18,
  "maximumAge": 65,
  "excludedCountries": ["PRK", "USA"],
  "includedCountries": ["IND", "JPN"],
  "ofac": true
}
```

| Field | Type | Notes |
| --- | --- | --- |
| `minimumAge` | integer | Age floor, 0 to 120. The user proves they are at least this old without revealing their birth date. |
| `maximumAge` | integer (optional) | Age ceiling, 0 to 120. Must be greater than or equal to `minimumAge`. Combine with `minimumAge` for a band. |
| `excludedCountries` | string array | Issuing countries to deny. ISO 3166-1 alpha-3 codes, sorted. |
| `includedCountries` | string array (optional) | Issuing countries to allow. Use one of include or exclude, not both. |
| `ofac` | boolean | When true, the user must clear the OFAC sanctions list. |

A Proof of Human flow has no rule fields; uniqueness is intrinsic to the document.

What the user's proof then attests to is a set of boolean outcomes, for example:

```json
{ "age_gte_18": true, "ofac_clear": true }
```

See [Disclosures](disclosures.md) for the full set of outcomes.

## Documents

Which credential types satisfy this flow. The user picks from the documents you allow:

* `biometric_passport`: ICAO 9303 e-passports. 60+ countries.
* `aadhaar`: India's national ID. See the [Aadhaar spec](../reference/document-specifications/aadhaar.md).
* `kyc_attestation`: partner-issued KYC. See the [KYC spec](../reference/document-specifications/kyc.md).

Documents map to capability. If a document cannot satisfy a rule (for example an age floor where the document does not carry a date of birth), the dashboard warns you before you publish.

## Settings

* **`displayName`**: what the user sees on the hosted page (for example "Acme Marketplace verification").
* **`successUrl`**: redirect on success.
* **`failureUrl`**: redirect on failure.
* **`branding`**: logo URL and accent color.

Per-session overrides for `successUrl` and `failureUrl` are accepted by the API, useful when the destination varies per user.

## Versions

A **flow** is the identity (the `flowId` you reference). A **flow version** is a frozen snapshot of `(product, rules, documents, settings)`. Every time you publish, a new version is created with a new `flowVersionId`.

* A session is pinned to a `flowVersionId` at creation time. It cannot move to a newer version mid-flight.
* The flow's `latestPublishedVersionId` is what new sessions use.
* Older versions are retained for audit and rollback.

## Lifecycle

```
draft ──publish──▶ published v1 ──edit──▶ draft ──publish──▶ published v2
                       │                                          │
                       └──── sessions still pinned to v1 ─────────┘
```

There is no explicit "archive a version". Archive happens at the flow level (`archivedAt`). An archived flow stops accepting new sessions but its history stays intact.

## Storage shape (for the curious)

If you integrate against the data model directly (for example via the dashboard's audit export):

```
flows
├─ id
├─ orgId
├─ product                   ← pre_kyc | age_verification | reusable_kyc
├─ name
├─ latestPublishedVersionId  ← FK to flow_versions
└─ archivedAt

flow_versions
├─ id
├─ flowId
├─ versionNumber
├─ configId                  ← FK to the config (the canonical predicate JSON)
└─ publishedAt
```

The `config` table holds the canonical predicate JSON, content addressed so identical configs dedupe. That is what the dashboard surfaces in the Configure tab.

{% hint style="info" %}
On the wire the three products carry the internal slugs `pre_kyc`, `age_verification`, and `reusable_kyc` (the dashboard labels `reusable_kyc` as Proof of Human). You only ever see these in an audit export. When creating sessions you reference a `flowId`, never the product slug.
{% endhint %}

## Related

* [Disclosures](disclosures.md): every outcome a proof can attest to.
* [Supported documents](supported-documents.md): what each credential type can prove.
* [Test vs. live](test-vs-live.md).
* [Configure a product](../dashboard/configure-a-product.md).
