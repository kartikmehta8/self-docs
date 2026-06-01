# Concepts

The model is small. Five things to know.

## Organization

The top-level tenant. Flows, API keys, webhook subscriptions, members, and billing all hang off an org. A user can belong to multiple orgs; you switch between them in the dashboard's top-left selector.

Created on sign-up. Renamed and deactivated under **Settings → Account**.

## Product

A product is the kind of verification you're running. There are three, and each one decides what the user has to prove and which configuration options you get:

* **Pre KYC**: a pre-screening check before full KYC. Confirms the user holds a genuine government document, optionally meets an age floor, comes from an allowed country, and clears the OFAC sanctions list. Outcome: approved or rejected.
* **Age Verification**: proves the user meets an age threshold (say 18 or 21) without revealing their birth date. Outcome: approved or rejected.
* **Proof of Human**: proves the user is a unique, real human backed by a government document, for sybil resistance. No age or country rules. Outcome: verified or failed.

You pick the product when you create a flow. Each product has its own per-verification cost; see [Plans](../billing/plans.md).

## Flow

A flow is a published, versioned configuration of one product. It pins:

* **Rules**: what the user must prove, drawn from the chosen product's options (an age floor, country allow or deny lists, OFAC).
* **Documents**: which credential types are acceptable.
* **Settings**: success URL, failure URL, branding.

Flows are versioned. Editing creates a draft; publishing freezes the draft as a new version and points the flow's `latestPublishedVersionId` at it. Older versions stay queryable for audit.

A `flowId` is what you pass to `sessions.create(...)`.

## Session (a.k.a. verification)

A session is a single attempt at verifying one user against one flow.

It has a lifecycle:

```
pending (created, awaiting the user) → valid | invalid | error | expired
```

* `valid`, proof verified and every predicate passed.
* `invalid`, proof verified but a predicate failed (e.g. underage).
* `error`, a technical failure (unsupported document, malformed proof).
* `expired`, the user never finished before `expiresAt`.

When a session reaches any terminal status, a `verification` record is finalized in your audit log with the proof attributes the user disclosed, and we fire the `verification.completed` webhook (its `status` field carries the outcome above).

Sessions also have a per-session **cost** in credits, baked in at creation time. See [Credits and usage](../billing/credits-and-usage.md).

## API key

A Bearer credential used by the Enterprise SDK to authenticate. Issued in two flavors:

* `sk_test_...`: test environment. Talks to test flows only. Mock passports accepted. No credits charged.
* `sk_live_...`: production. Real proofs. Real credits.

Keys are scoped to an org. Issue, rotate, and revoke under **Settings → API keys**.

## Webhook subscription

A signed delivery channel for events. You register a URL in **Settings → Webhooks** and pick which event types you want. We send signed JSON payloads with automatic retries.

See the [event catalog](../webhooks/events.md) for what's available.

## Related concepts

* [How verification works](how-it-works.md): the zero-knowledge model these objects sit on top of.
* [Anatomy of a flow](../flows/anatomy.md): deep dive into rules, documents, and settings.
* [Dashboard: API keys](../dashboard/api-keys.md): how Bearer keys map to environments.
* [Billing](../billing/credits-and-usage.md): credits, plans, metering.
