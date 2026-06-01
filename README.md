---
icon: hand-wave
cover: .gitbook/assets/110ft QuickZip Straight.png
coverY: 0
description: Verify real-world identity in your product, without ever holding the data.
---

# Welcome to Self

Self verifies real-world identity using **zero-knowledge proofs**. Your users prove a fact about themselves, they're over 18, they're a unique human, they're not on a sanctions list, and you get a yes/no answer. You never see their passport, their date of birth, or their name.

**[Self Enterprise](self-enterprise/get-started/what-is-self-enterprise.md)** is the fastest way to use it: a dashboard, one SDK call, and a signed webhook. No smart contracts, no verifier to host, no PII to store.

{% hint style="success" %}
**New here?** Go from zero to a verified user in ten minutes with the **[Quickstart](self-enterprise/get-started/quickstart.md)**.
{% endhint %}

## Start with Self Enterprise

The managed platform. You set up a verification flow in the dashboard, ask Self to start a verification when a user needs to prove something, and hand them a link. When they finish, Self sends your backend a signed message with the result. No smart contracts, no verifier to host, no personal data to store.

<table data-view="cards"><thead><tr><th></th><th></th><th data-hidden data-card-target data-type="content-ref"></th></tr></thead><tbody><tr><td><strong>Quickstart</strong></td><td>Verified user in ten minutes.</td><td><a href="self-enterprise/get-started/quickstart.md">quickstart.md</a></td></tr><tr><td><strong>Core concepts</strong></td><td>Orgs, flows, sessions, keys, webhooks.</td><td><a href="self-enterprise/get-started/concepts.md">concepts.md</a></td></tr><tr><td><strong>How verification works</strong></td><td>The zero-knowledge model, in plain language.</td><td><a href="self-enterprise/get-started/how-it-works.md">how-it-works.md</a></td></tr><tr><td><strong>SDK</strong></td><td>The <code>@selfxyz/enterprise-sdk</code> client.</td><td><a href="self-enterprise/sdk/nodejs.md">nodejs.md</a></td></tr><tr><td><strong>Webhooks</strong></td><td>Signed events, signature verification, idempotency.</td><td><a href="self-enterprise/webhooks/overview.md">overview.md</a></td></tr></tbody></table>

## What you can build

* **Sybil resistance**: admit only unique, real humans, without collecting personal data.
* **Age gates**: restrict access by age without ever learning a date of birth.
* **Geographic and sanctions compliance**: allow or deny by nationality, screen against OFAC.
* **Reusable KYC**: verify a user once, trust the attestation later.
* **Marketplace trust**: verify both sides of a trade without holding either party's PII.

## More from Self

Self builds more than verification. These products have their own sections in the sidebar:

* **[Self Connect](self-connect/introduction-and-overview.md)**: map off-chain identifiers (phone, email, social handles) to on-chain addresses.
* **[Self Agent ID](agent-id/overview.md)**: on-chain proof-of-human identity for AI agents (ERC-8004).

## Resources

* [Dashboard](https://dashboard.self.xyz): sign up and configure your first flow.
* [Interactive coverage map](https://map.self.xyz/): which documents and countries are supported.
* [Self Builder Group](https://t.me/+d2TGsbkSDmgzODVi): Telegram community for developers.
* [Status](https://status.self.xyz): live service status.
