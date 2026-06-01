---
icon: shield-halved
description: The zero-knowledge model behind Self Enterprise, in plain language.
---

# How verification works

You never have to touch the cryptography, the SDK and webhooks hide it. But the model is worth a minute, especially for reasoning about privacy. For the full cryptographic detail, see the [Self Pass protocol docs](../../self-pass/architecture/zk-proof-architecture.md).

## The idea in one line

The user's phone proves a **fact** about their government ID ("over 18", "not sanctioned"), and Self checks that proof. You get the fact, never the underlying data.

## The flow

```
        ┌──────────────────┐
        │   User's phone   │   reads the ID chip and builds
        │    (Self app)    │   a ZK proof on the device
        └────────┬─────────┘
                 │  only the proof leaves (a fact, not the data)
                 ▼
        ┌──────────────────┐
        │       Self       │   verifies the proof
        └────────┬─────────┘
                 │  the fact, e.g. "over 18: yes"  (webhook)
                 ▼
        ┌──────────────────┐
        │   Your backend   │
        └──────────────────┘
```

1. The user reads their ID chip with the Self app, **once**. The credential then lives on their device.
2. For each verification, the app builds a fresh **zero-knowledge proof** of exactly what your flow asks, nothing more.
3. Self verifies the proof and sends your backend the result. The raw ID data never leaves the phone.

{% hint style="info" %}
The document is signed by the issuing government. Self maintains the certificate registry that makes a proof trustworthy, so a forged or unsupported document simply fails. See [supported countries](../reference/supported-countries.md).
{% endhint %}

## What you get vs. what you never see

| You receive | You never see |
| --- | --- |
| Predicate results (`age_gte_18: true`) | Date of birth |
| Any explicit reveal you asked for | Name |
| A stable per-user nullifier (uniqueness) | Passport or document number |
| The document type used | The chip data or biometrics |
| A verification ID for your audit log | Anything you didn't ask for |

This is the whole privacy guarantee: **the proof attests to exactly what your flow asks, and nothing else leaks.** See [Disclosures](../flows/disclosures.md#what-is-never-disclosed).

## One account per person (nullifiers)

Need to stop duplicates? Each verification carries a **nullifier**: a value derived from the document that is the same for the same person in your product (so you can catch repeats), reveals nothing about who they are, and is different and unlinkable in every other product (so users can't be tracked across services).

## Going deeper

The circuits, the commitment scheme, the certificate trees, and the on-chain verification hub are documented in the legacy Self Pass section:

* [ZK Proof Architecture](../../self-pass/architecture/zk-proof-architecture.md)
* [Verification in the IdentityVerificationHub](../../self-pass/architecture/verification-hub.md)
* [OFAC & CSCA Auto-Updaters](../../self-pass/architecture/ofac-csca-auto-updaters.md)

## Next

* [Core concepts](concepts.md): the objects you work with.
* [Quickstart](quickstart.md): wire it up end-to-end.
* [Disclosures](../flows/disclosures.md): everything you can ask users to prove.
