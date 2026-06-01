# What is Self Enterprise

Self Enterprise is the managed plane for zero-knowledge identity verification. You configure flows in a dashboard, call a single SDK, and receive signed webhook events when users verify. We handle credential issuance, proof verification, decentralized storage, and the audit trail.

The integration is small: one `sessions.create(...)` call, a hosted page the user opens, and a webhook on your backend.

## What it replaces

Self Enterprise is the managed offering on top of the open [Self protocol](../../open-source/README.md). It bundles the parts you'd otherwise build and operate yourself:

| Concern | Open-source Self | Self Enterprise |
| --- | --- | --- |
| Configure a verification flow | Encode disclosures in your contract / backend | Configure in the dashboard, publish a version |
| Issue credentials to users | Run your own backend + hosted page | Hand the user a `verificationUrl` from the API |
| Verify a proof | Run the verifier yourself (or on-chain) | We verify on the hot path; you get a signed webhook |
| Store the result | Build your own store + audit log | Optional decentralized storage; full audit log in the dashboard |
| Bill the work | n/a | Usage-based, metered automatically |

If your team wants a fast path from "we want ZK identity verification" to a working integration, Enterprise is it. If you'd rather own the full stack, the [open-source SDKs](../../open-source/README.md) are still the right tool.

## Who is it for?

* **Consumer apps** running sybil resistance, age gates, or geographic compliance. Anyone who needs verified identity but doesn't want to build verification infrastructure.
* **Fintech / regulated platforms** that need KYC-grade verification with auditable trails.
* **Marketplaces** that want to verify both sides of a trade without holding PII.
* **Internal compliance teams** at protocols who'd otherwise hand-roll a verifier.

## How it fits together

```
┌──────────────┐    sessions.create()    ┌──────────────┐
│ Your backend │ ─────────────────────▶  │  Self        │
└──────────────┘                          └──────┬───────┘
       ▲                                          │
       │ webhook (verification.completed)         │ hosted
       │                                          ▼
┌──────────────┐                          ┌──────────────┐
│ Your backend │  ◀──────────────────────  │  Self app   │
└──────────────┘   signed webhook         │  (user)      │
                                          └──────────────┘
```

1. **You configure a flow** in the dashboard (what to verify: age, nationality, sanctions, etc.).
2. **You call `sessions.create(...)`** via the Enterprise SDK when a user needs to verify. We return a `verificationUrl`.
3. **Your user opens the URL** in their Self app and produces a ZK proof of the requested attributes.
4. **We verify the proof** on the hot path and fire a webhook to your backend.
5. **You read the verified attributes** from the webhook and proceed.

Next: [Quickstart](quickstart.md) walks the end-to-end integration in ten minutes.
