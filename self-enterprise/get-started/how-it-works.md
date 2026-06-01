---
icon: shield-halved
description: The zero-knowledge model behind Self Enterprise, in plain language.
---

# How verification works

You don't need to understand zero-knowledge cryptography to use Self Enterprise, the SDK and webhooks hide all of it. But it helps to know what's happening, especially when you're reasoning about privacy and trust. This page explains the model in plain language. For the cryptographic depth, the [open-source protocol docs](../../self-pass/architecture/zk-proof-architecture.md) have the full detail.

## The one-sentence version

The user proves a **fact** about their government ID (e.g. "I'm over 18") to their phone, the phone produces a **zero-knowledge proof** of that fact, and Self verifies the proof. You learn the fact, never the ID.

## The three actors

```
┌─────────────┐   1. create session    ┌──────────────┐
│ Your backend │ ─────────────────────▶ │     Self      │
└─────────────┘                          └──────┬───────┘
       ▲                                         │ 2. hosted page
       │ 4. signed webhook                       ▼
       │    (the fact)                    ┌──────────────┐
       └──────────────────────────────── │  Self app    │
                                          │  (the user)  │
                                          └──────────────┘
                                          3. scan ID, prove on-device
```

1. **Your backend** asks Self for a verification session and gets back a URL.
2. **Self's hosted page** opens in the user's Self app.
3. **The user's phone** reads their passport/ID chip and generates a proof locally. The raw document data never leaves the device.
4. **Self verifies the proof** and sends your backend a signed webhook with the disclosed facts.

## Step 1: Registration (one time per user)

Before a user can prove anything, their identity document has to be turned into a credential. They scan their passport's NFC chip (or Aadhaar QR) once with the Self app.

The chip's data is signed by the issuing government using a **Document Signer Certificate (DSC)**, which in turn chains up to that country's **Country Signing Certificate Authority (CSCA)**. Self maintains a registry of these certificates, that's how a proof can be trusted to come from a genuine, government-issued document. (Self auto-updates this registry; see [supported countries](../reference/supported-countries.md).)

This registration step produces a **commitment**, a cryptographic fingerprint of the document, that lives in the Self protocol. The heavy proof here is generated in a secure environment; subsequent disclosure proofs are cheap and run on the device.

{% hint style="info" %}
Registration happens inside the Self app, transparently to your integration. You never handle it.
{% endhint %}

## Step 2: Disclosure (every verification)

When the user opens one of your verification links, their app generates a fresh **zero-knowledge proof** that answers exactly the questions your flow asks, and nothing else:

* *"Is this person at least 18?"* → proof says **yes**, without revealing their birth date.
* *"Is their nationality outside this list?"* → proof says **yes**, without revealing which country.
* *"Are they on the OFAC sanctions list?"* → proof says **no**, without revealing their name.

These are **predicates**: true/false answers computed from the underlying data without disclosing the data itself. (You can also request a small number of explicit **reveals**, like exact nationality, see [Disclosures](../flows/disclosures.md).)

## Step 3: Verification and the result

Self checks the proof against:

* the government certificate chain (is the document real?),
* the flow's rules (does the user satisfy your predicates?),
* and a **uniqueness check**.

If it all holds, you get a `verification.completed` webhook with `status: "valid"` and the disclosed `proof_attributes`. See the [event catalog](../webhooks/events.md).

## Uniqueness without identity (nullifiers)

A common need is "one account per real person." Self provides this with a **nullifier**, a deterministic value derived from the document that is *stable for the same person* but *reveals nothing about who they are*.

Because the nullifier is scoped per application, the same person produces the same nullifier in your product (so you can detect duplicates) but a *different, unlinkable* nullifier in someone else's product (so your users can't be tracked across services).

## What you receive vs. what you never see

| You receive | You never see |
| --- | --- |
| Predicate results (`age_gte_18: true`) | Date of birth |
| Explicit reveals you asked for (e.g. nationality) | Name |
| A stable per-user nullifier (uniqueness) | Passport number |
| The document type used (passport / Aadhaar / KYC) | The chip data or biometric |
| A verification ID for your audit log | Anything you didn't request |

This is the core privacy guarantee: **the proof attests to exactly what your flow's rules ask, and nothing else leaks.** See [Disclosures → What's never disclosed](../flows/disclosures.md#whats-never-disclosed).

## Why this is better than collecting documents

* **No PII liability.** You can't leak what you never hold. There's no passport scan in your database to breach.
* **No verifier to run.** Self verifies proofs on the hot path and delivers the result.
* **Auditable.** Every verification is recorded against the exact flow version it ran on, see the [Activity log](../dashboard/activity-log.md).

## Going deeper

The full cryptographic design, circuits, the commitment scheme, the two kinds of nullifier, the certificate trees, and the on-chain verification hub, is documented in the open-source protocol section:

* [ZK Proof Architecture](../../self-pass/architecture/zk-proof-architecture.md)
* [Verification in the IdentityVerificationHub](../../self-pass/architecture/verification-hub.md)
* [OFAC & CSCA Auto-Updaters](../../self-pass/architecture/ofac-csca-auto-updaters.md)

## Next

* [Core concepts](concepts.md), the objects you'll actually work with.
* [Quickstart](quickstart.md), wire it up end-to-end.
* [Disclosures](../flows/disclosures.md), the full catalog of what you can ask users to prove.
